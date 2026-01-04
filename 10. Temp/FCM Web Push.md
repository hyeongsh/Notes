---
tags:
  - BetterLife
  - Backend
  - FCM
  - Redis
---
FCM을 이용한 WebPush 방식은 타이머 종료 알림, 일정 알림, 리마인더 알림 등 웹에서 네이티브 앱처럼 울리도록 할 수 있는 방식이다.
여기서 중요한 점은 웹사이트에서도 모바일 앱처럼 알림을 받을 수 있다는 것이다.
Service Worker 와 FCM WebPush 를 사용하여 이 방식을 구현할 수 있다.
## FCM WebPush 전체 구조
1. Web Brower에 Service Worker가 붙는다.
2. FCM 서버에서 브라우저에 알림 권한을 요청하여 웹 푸시 토큰을 받는다.
3. 백엔드 서버에서 FCM 서버로 푸시 메시지를 요청한다.
4. FCM 서버에서 발송 시간까지 기다리다가 푸시를 발송한다.
### 프론트엔드 알림 권한 요청
알림을 허용하시겠습니까?
Notification.requestPermission()
## Firebase SDK로 Push Token 발급
const token = await getToken(messaging, { vapidKey: "공개키;"
});
브라우저 고유 Push Token 획득
### 서버에 토큰 저장
POST /notification/token
{
	"userId": 15,
	"token": "공개키"
}
Redis나 RDB 어느 곳에든 저장한다.
### 특정 시각에 알림이 필요한 경우
Notification Worker가 endTime에 맞춰 조회한다.
여기서 Redis에 알림을 넣고 초마다 조회함으로써 DB 탐색 시간을 줄인다.
self.addEventListener("push", (event) => {
	const data = event.data.json();
	self.registration.showNotification(data.tile, {
		body: data.body,
		icon: "/icon.png",
	});
});

## 프론트에서 토큰 발급 개념
사용자 브라우저로부터 알림 권한을 얻는 과정이다.
브라우저는 기본적으로 웹사이트에 푸시 알림을 허용하지않기 때문에 사용자가 명시적으로 알림을 허용해야 한다.

브라우저가 권한을 허용하는 것과 별개로 브라우저마다 FCM에게 고유한 Push Token을 발급 받아야 한다.

중요한 점은 알림 허용과 별개인 점으로, 토큰은 계속해서 갱신해주어야 한다는 점이다.

그래서 로그인한 사용자 + 브라우저 조합별로 토큰을 저장해야 하며, 여러 기기에서 로그인한 경우에는 여러 토큰을 저장해야 하고 로그아웃을 할 경우 토큰까지 같이 삭제되어야 한다.
리프레쉬 토큰을 저장하듯 저장하는 방식을 사용해야 할듯
## 토큰 갱신 전략
웹 푸시 토큰은 언제든지 바뀔 수 있고, 토큰이 오래된 경우 알림을 받지 못하며, FCM이 토큰을 강제로 무효화하는 경우, 여러 브라우저에서 중복 로그인하는 경우 토큰이 매우 복잡하게 다루어지기 때문에 토큰 갱신 전략이 반드시 필요하다.
1. 프론트에서 갱신 발생 시 감지
2. 백엔드에서 갱신 요청을 받아 저장
3. 알림 발송 과정에서 무효 토큰 자동 정리
프론트에서는 토큰이 바뀌는 경우를 대비해야 한다.
브라우저 쿠키 삭제, 캐시 삭제, service worker 업데이트, firebase 내부 정책 등 여러가지 이유로 변경될 수 있다.
따라서 토큰이 변경된 경우 즉시 서버에 업데이트해야 한다.
### 앱 로드 시 getToken() 요청
앱이 열릴 때마다 getToken()을 요청하여 현재 토큰을 확인한 후 서버에 저장된 토큰과 다르다면 갱신한다.
### 로그인 / 로그아웃 시 서버 토큰도 업데이트
로그인 시 새 토큰을 서버에 저장하고, 로그아웃 할 경우 해당 브라우저 토큰을 서버에서 삭제한다.
### 백엔드 토큰 저장 전략
항상 최신 토큰만을 DB에 저장할 수 있어야 한다.
```
user_id  | token      | device | last_used_at | created_at

-----------------------------------------------------------

15       | abc123     | Chrome | 2025-12-15   | ...

15       | def456     | iOS-SF | 2025-12-15   | ...
```
토큰이 없을 경우 -> insert
토큰이 등록되어 있을 경우 -> last_used_at 갱신
브라우저가 기존 토큰과 다르다면 새 insert

1. fcm 형식에 맞게 메시지를 생성한다.
2. 헤더에 fcm access token을 넣는다.
3. fcm 프로젝트에 메시지를 전송한다.
  