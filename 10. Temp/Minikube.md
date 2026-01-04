# MiniKube Tutorials
## Objectives
- 미니큐브에 샘플 애플리케이션 올리기
- 앱 실행시키기
- 서비스 로그 보기
## 미니큐브 클러스터 생성
`minikube start` 로 미니큐브 클러스터를 생성한다.
다른 터미널을 열어 `minikube dashboard` 를 실행시키면 GUI로 클러스터를 관리하는 브라우저를 열 수 있다.
다시 시작명령을 내린 터미널로 돌아온다.
## Deployment 생성
쿠버네티스의 파드는 하나 이상의 컨테이너의 그룹을 말한다.
한꺼번에 관리하고 네트워킹할 수 있도록 되어있다.
하지만 이 튜토리얼에서는 하나의 컨테이너 당 하나의 파드로 구성될 것이다.
Deployment는 파드를 헬스 체크하고 만약 컨테이너가 중단됐을 경우 다시 시작시킨다.
```shell
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- /agnhost netexec --http-port=8080
```
## 서비스 생성
현재 상태는 내부 IP 주소만 접근이 되도록 되어있다.
hello-node 컨테이너가 밖에서도 접속 가능하도록 하려면 pod를 service로 expose해야한다.
```shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080

minikube service hello-node
```

### database 배포
kubectl create deployment mysql-quote --image=mysql:8 --port=3306  
kubectl set env deployment/mysql-quote MYSQL_ROOT_PASSWORD=root MYSQL_DATABASE=quote-db
kubectl expose deployment mysql-quote --name=mysql-quote --port=3306 --target-port=3306
### backend 배포
./gradlew clean bootJar
docker build -t shinhs6384/cp-subject-backend:latest .
docker push shinhs6384/cp-subject-backend:latest
kubectl create deployment cp-subject-backend --image=shinhs6384/cp-subject-backend:latest
kubectl expose deployment cp-subject-backend --name=cp-subject-backend --port=8090 --target-port=8090
minikube service cp-subject-backend --url
### frontend 배포
docker build -t shinhs6384/cp-subject-frontend:latest .
docker push shinhs6384/cp-subject-frontend:latest   
kubectl create deployment cp-subject-frontend --image=shinhs6384/cp-subject-frontend:latest
kubectl expose deployment cp-subject-frontend --name=cp-subject-frontend --port=80 --target-port=80 --type=NodePort
minikube service cp-subject-frontend --url
### ingress 추가
minikube addons enable ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cp-subject
spec:
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: cp-subject-backend
                port:
                  number: 8090
          - path: /
            pathType: Prefix
            backend:
              service:
                name: cp-subject-frontend
                port:
                  number: 80

```
kubectl apply -f ingress.yml

인그레스는 클러스터 외부에서 클러스터 내부 서비스로 HTTP와 HTTPS 경로를 노출한다. 트래픽 라우팅은 인그레스 리소스에 정의된 규칙에 의해 컨트롤된다.
다음은 인그레스가 모든 트래픽을 하나의 서비스로 보내는 간단한 예시이다.