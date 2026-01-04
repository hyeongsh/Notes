---
tags:
  - RabbitMQ
  - Messaging
  - Backend
  - Server
  - MSA
---
# 필수 요소
## ConnectionFactory
RabbitMQ 서버와 연결을 생성하는 역할
직접 만들 필요는 없으며 spring.rabbitmq.* 설정을 확인하고 이를 기반으로 자동 생성한다.
## RabbitTemplate (Producer)
서비스에서 메시지를 발행할 때 사용하는 RabbitMQ 전송 도구이다.
`rabbitTemplate.convertAndSend(exchange, routingKey, message);`
RabbitTemplate은 대부분의 서비스에서 Producer 역할을 하므로 핵심이다.
Spring Boot에서는 이걸 자동으로 빈으로 만들어준다. 하지만 메시지 변환기를 설정하기 위해 직접 생성해야 하는 경우도 존재한다.
### 이벤트 설계 베스트 프렉티스
1. 이벤트 이름은 과거형으로 작성한다.
2. 이벤트는 사실만 전달하고 명령을 전달하지 않는다.
3. 이벤트 스키마에는 필요한 정보만 포함한다.
4. 이벤트는 절대 변경되지 않도록 설계한다.
5. 이벤트 버전을 반드시 설계한다.
6. 이벤트는 서비스 간 의존성을 최소화하도록 설계해야 한다.
7. Exchange와 RoutingKey Naming 규칙을 일관되게 한다.
8. 이벤트는 단순 JSON일수록 좋다.
9. 실패 처리와 재시도 전략을 반드시 설계한다.
10. 이벤트의 발행 시점은 트랜잭션이 성공적으로 끝난 직후 전송한다.