# GPT 답변
S3의 역할은 정적 파일 저장소이다.
S3는 말그대로 파일을 저장하는 공간으로 React의 빌드 결과물이 올라가도록 구현되었다.
단순히 파일을 저장하는 서버 역할을 하며, 서버가 파일을 직접 서비스하지 않으며 원본 저장소로만 사용이 된다.
사용자 요청이 직접 S3에 가는 구조는 성능이 좋지 않으므로 CloudFront가 앞단에서 캐싱과 전송을 담당한다.
CloudFront는 S3 앞에 있는 Content Delivery Network(CDN)이다.
전세계 엣지 로케이션에 캐싱을 해두기 때문에 한국, 미국, 일본 어디서 접근하든 가장 가까운 엣지에서 파일을 전달해주며, S3는 자체적으로 HTTPS 커스텀 도메인을 지원하지 않기 때문에 CloudFront가 ACM 인증서를 연결하여 도메인 형태로 제공해줄 수 있다.
도메인은 S3에 연결되는게 아니라 CloudFront 배포에 연결된다.
s3를 생성할 때 public access off (OAC) 방식으로 생성하였고, CloudFront에 OAC를 연결한 후, 밖에서 CloudFront로 접속하는 경우엔 HTTPS 인증서를 통해 HTTPS 인증을 처리하였다.
### 사용자 요청이 직접 S3에 가는 구조가 성능이 좋지 못한 이유
s3는 단순한 저장소이며, 트래픽을 빠르게 전달하기 위한 전송 최적화 계층이 아니다.
즉 S3에 직접 요청하는 구조는 지리적 구조로 인한 Latency가 매우 크다.
캐싱을 할 곳도 없이 원본 위치에서 전부 전송해줘야 하므로 너무 오래 걸린다는 문제가 있다.
S3에서 나가는 트래픽 비용은 CloudFront보다 훨씬 비싸다.
S3는 지원하는 HTTPS가 https://bucket-name.s3.amazonaws.com/ 이런 형태뿐이고 커스텀 도메인 연결은 불가능하다.
#### 구현 시에...
홈서버에 구축할 경우 CDN 캐싱 방식은 안될 것 같고 트래픽을 빠르게 전달해주는 용도로 nginx가 잘 해줄까 싶다.
#### TLS 오프로드
SSL은 조금 더 안전한 통신 방식이다.
SSL 통신을 위해 초기에 클라이언트와 서버 사이에 SSL HandShaking을 하게 된다.
이 작업이 생각보다 많은 resource를 사용하게 된다.
1. 클라이언트가 ClientHello를 보낸다.
	1-1) SSL/TLS version  
	1-2) SSL 통신에 사용할 암호 알고리즘  
	1-3) 데이터 압축 방법 (ex Gzip)
2. 서버가 ServerHello를 보낸다.
	2-1) 클라이언트가 요청한 암호 알고리즘의 대한 동의  
	2-2) 서버에서 생성한 SessionID  
	2-3) 서버의 digital certificate  
	2-4) 서버의 public key
3. 클라이언트 측에서 digital certificate를 CA에서 확인하는 절차를 거친다.
4. 클라이언트에서 서버가 보내줬던 public key를 이용해 shared secret key를 만들어 공유한다.
5. 클라이언트 측에서 먼저 finish를 보내고 서버도 finish를 보내 핸드셰이크가 끝났음을 알린다.
이걸 API 서버가 관리하도록 하면 너무 큰 작업을 담당하게 되므로 서버의 성능 상 좋지 않다.
그래서 중간에 Proxy 서버를 두어 SSL 쪽 성능이 뛰어난 서버가 중간에서 HTTPS 처리를 완료하고 로드 밸런싱을 통해 적절한 API 서버에 HTTP로 던져주는 방식을 사용하게 된다.
즉, S3 서버가 TLS에 대한 부담이 없어지므로 조금 더 값싸게 사용할 수 다는 뜻이다.
### OAC의 자세한 설명
OAC는 CloudFront가 도입한 Origin Access Control 기능이다.
CloudFront만이 S3에 접속할 수 있도록 만드는 역할을 한다.
OAC를 적용한 순간 S3 버킷에 public으로 접근하는 것을 완전히 차단하고 CloudFront와는 서명을 포함한 요청을 통해 보안을 유지하게 된다.
### CloudFront에 인증서를 연결한 ACM 인증서의 역할
CloudFront는 HTTPS 제공을 위해 반드시 SSL 인증서가 필요한데, AWS의 ACM에서 관리하게 된다.
Let's Encrypt과 같은 목적이지만 차이는 있다.
#### Let's Encrypt
인증서를 서버에 직접 설치해야 하며 웹 서버가 TLS hadshake를 처리하는 방식이다.
User -> HTTPS -> Nginx -> App
#### ACM
인증서를 AWS 리소스에 붙이는 방식이다.
서버에 설치되지 않고 CloudFront에 적용하는 방식이다.
AWS Edge 네트워크가 TLS handshake를 처리하게 된다.
User -> CloudFront -> S3
