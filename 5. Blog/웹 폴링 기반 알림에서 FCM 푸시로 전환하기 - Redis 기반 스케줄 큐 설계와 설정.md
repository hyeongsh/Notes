## 기존 방식
내 개인 프로젝트에서 알림을 폴링 방식을 이용해 받는 형식으로 사용하고 있었다.
이 방식을 사용할 경우 몇 가지 장점이 생기는데,
1. 외부 서비스를 사용할 필요가 없다.
2. 코드가 간단해진다.
정도로 정리할 수 있다.
단점은 더 간단하다. 트래픽 낭비가 심하다는 것이다.
그럼에도 이 폴링 방식 알림을 사용한 이유는 간단하다.
todo에 대한 알림이니 5분마다 한번씩 확인해도 충분하기에 5분 간격의 폴링은 통신 부담이 없었다.
그러다가 타이머를 도입하기 시작하면서 문제가 발생했다.

---
## 타이머는 어떤 식으로 처리할건데?
프론트를 잘 알지 못하다보니 생긴 문제가 타이머를 어떻게 처리하는게 베스트냐는 것이었다.
1초마다 폴링 방식을 사용하기엔 너무 벅차보였고, 그렇다고 타이머를 전적으로 프론트에 맡기기에는 웹을 나갔을 때의 문제를 해결할 수 없었다.
**사실 따지고 보면 웹에서 타이머를 돌리는 부분에서 웹을 나가는 것까지 신경쓰는 것이 맞나..** 라는 고민을 수십 번 하였다.
하지만 어차피 같은 백엔드로 웹을 넘어 앱까지 구현할 예정이었기에, 알림 방식의 통일은 언젠가 해야할 일이었고, 그 방식으로 가장 좋은 방식은 FCM 푸시 방식이었기에 곧장 시도하게 되었다.

---
## FCM 설정
FCM HTTP v1 API를 사용하여 구현하였다.
프로젝트 전체에서 OAuth2 기반 인증을 사용하는 방식으로 서버에서 OAuth2 access token을 발급하면 같은 서버에서는 계속 사용 가능한 방식이다.
서버 쪽에서는 GoogleCredentials 로 서비스 계정을 로딩하여, https://fcm.googleapis.com/v1/projects/{project-id}/messages:send 엔드포인트로 전송하는 방식을 사용하였다.
**message 포맷**은 여러가지 방식이 사용가능해서 고민했으나 최종적으로 사용한 방식은 token, data(title, body, link), webpush(headers) 정도만 담아 최소한으로 사용하고자 하였다.
원래 notification이라는 필드도 존재하는데 이 필드는 webpush 안에도 notification이 들어가 중복 부분에서 설정이 복잡해졌다.
그래서 복잡한 부분이 추가될수록 어지러워지는 것 같아 바로 패스했다.

## fcm 설정법 상세
### 1) Firebase 프로젝트 준비
1. Firebase Console → 새 프로젝트 생성
2. Project Settings → Cloud Messaging 활성화 확인

---
### 2) Web 앱 등록 (웹푸시 토큰 발급용)
1. Project Settings → General → “웹 앱 추가”
2. 앱 이름 등록 후 firebaseConfig 값 확보
    - apiKey, projectId, messagingSenderId, appId, authDomain 등

---
### 3) VAPID 키 생성 (웹푸시 전용)
1. Project Settings → Cloud Messaging → Web Push certificates
2. “Generate Key Pair”
3. **Public VAPID Key**를 프론트 토큰 발급에 사용

---
### 4) 서비스 계정 키 (서버 인증)
1. Project Settings → Service Accounts
2. “Generate new private key”
3. JSON 파일 다운로드
    - 서버에서 OAuth2 토큰 발급용으로 사용

---
### 5) 서버 인증 설정 (HTTP v1)
서버에서는 서비스 계정 JSON으로 액세스 토큰을 발급하고, 아래 scope를 사용한다.
- 권장 scope:  
    https://www.googleapis.com/auth/firebase.messaging
- 전송 URL:  
    https://fcm.googleapis.com/v1/projects/{project-id}/messages:send
Spring 기준:
- GoogleCredentials.fromStream(서비스계정json).createScoped(...)
- googleCredentials.refreshIfExpired()
- Authorization: Bearer {access_token}

---
## Redis Only
우선 RDB를 쓰지않고 Redis를 쓴 이유는 매초마다 쿼리를 날릴 경우 범위 조회에 대한 시간 문제를 감당할 수 없을 것 같다는 확신(?)이 들어서 였다.
찾아보다가 이를 위해 제공하는 Redis ZSET 기능을 발견하였다.
sorted set이며 score에 대한 매핑 값을 저장할 수 있는 기능을 가지고 있다.
1. score를 timestamp로 두면 범위 조회가 빠르다.
2. due time 기준으로 선별이 가능하며 구조가 단순하다.
3. 데이터 접근 비용이 낮다.
이 장점들을 이유로 redis만을 이용해 알림을 구현하고자 하였다.
(물론 fcm token은 계속 저장될 필요가 있어서 DB에 저장했다.)
그러다 보니 또다른 문제가 발생했는데,
4. 레디스에는 string 값으로 들어가니, 가져올때마다 readValue 단계에서 예외처리를 해줘야한다는 점
5. todo나 user 삭제 시 관련된 알림을 삭제해야 하는데, 이를 위한 편리하게 제공되는 메서드가 존재하지 않으므로 todoId, userId 에 해당하는 인덱스를 직접 만들어줘야 한다는 점
6. 데이터 정합성을 온전히 보장해줄 수 없고, DB의 트랜잭션 기능을 활용할 수 없다는 점
등이 문제가 되었다.
이를 위해 DB에 알림을 저장하고 레디스는 매초 score를 비교하는 용도로만 사용하는 방식도 고려해보았으나, 알림의 생성량이 많으며, 저장을 해둘 필요가 없고, 쿼리 속도를 감당할 가치가 없음을 이유로 레디스 온니를 고집하였다.
내가 예외처리한 부분은 로직부분에서의 예외만 처리했으며, 예외가 발생할 경우 (데이터 정합성이 맞지 않음, 전송이 실패함(fcm 토큰 만료), readValue 실패) 해당 알림을 폐기하고 다른 알림은 다음 주기에서 재시도한다는 기준으로 처리하였다.
예외는 항상 워커가 받아서 처리하도록 하여 매초 도는 로직을 방해하지 않도록 하려고 노력했다.

---
이로 인해 개선된 사항은 다음과 같다.
1. 트래픽 절감
	1. 폴링 사용 시 사용자 1만 명일 경우 5초 폴링이 걸리면 분당 12만 개 요청을 "반드시" 해야 했다.
	2. 푸시로 변경 후 실제 알림 발생 수만큼만 트래픽이 발생하여 API와 DB 부담을 줄였다.
2. 배터리/데이터 사용량 감소
	1. 폴링 사용 시 앱/브라우저가 지속적으로 네트워크를 사용하였다.
	2. 푸시로 변경 후 데이터 사용량을 감소하였다.