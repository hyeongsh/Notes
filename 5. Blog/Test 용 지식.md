---
tags:
  - Backend
  - BetterLife
  - Test
---
### ReflectionTestUtils
Spring Test 모듈에 들어있는 유틸 클래스로 리플렉션을 이용해 private 필드 값을 주입하고 변경하며, private 메서드를 호출하고 static 필드/메서드 또한 조작 가능하도록 도와준다.
보통 DI가 안 걸린 순수 객체에 private 필드를 주입해야 하는 경우나 레거시 코드라 setter가 없는데 테스트에서만 값을 넣고 싶을 경우에 사용한다.
보통 spring-boot-starter-test에 같이 들어온다.
### MockServerHttpRequest
Spring WebFlux 에서 사용하는 ServerHttpRequest를 테스트용으로 쉽게 만들어주는 Mock의 구현체다.
`org.springframework.mock.http.server.reactive.MockServereHttpRequest` 클래스
보통 컨트롤러/핸들러/필터 테스트에서 실제 서버 없이 요청 객체를 구성해서 넣어보는 용도로 사용한다.
WebFlux에서는 요청이 HttpServletRequest가 아니라 리액티브 타입의 ServerHttpRequest로 흐르기 때문에 WebFlux 용 mock를 따로 사용하는 것.
이를 이용해 URI / query, headers, cookies, remoteAddress 등을 설정한다.
ServerWebExchange가 필요하여 같이 만드는 용도다.
핵심은 리액티브 요청 바디까지 포함한 요청을 테스트에서 만들 수 있다는 점
### ServerWebExchange
WebFlux에서 요청 1건을 처리하는 한 묶음 이라고 보면 된다.
exchange 안에는 request, response, attributes, 기타 가 들어간다.
그 전에 체인은 WebFlux에서 요청을 받을 때 필터 여러 개를 거쳐서 최종 핸들러나 컨트롤러로 들어오게 되는데, 여기서 다음 단계로 넘겨달라는 뜻이 chain.filter(exchange)다.
즉, 체인은 다음 필터를 호출하는 통로를 의미한다.
그러므로 체인에 전달된 exchange라는 뜻은 다음 단계로 넘기는 요청 컨텍스트를 의미하며, 현재 필터를 거치는 과정에서 만약 무언가가 수정되었다면 해당 수정을 확인해보겠다는 뜻이다.
### AtomicReference\<ServerWebExchange>
람다/익명 클래스 안에서 바깥 변수에 값을 저장하려고 값을 담는 상자로 만들어 둔 것
Java에서 람다 안에서는 바깥 지역변수를 수정할 수 없기 때문에, 테스트에서 필터/체인 안에 들어온 exchange를 밖으로 꺼내 확인하고 싶을때 AtomicReference를 많이 쓴다.
예를 들어 WebFilter나 GlobalFilter 테스트 같은 경우에 체인으로 exchange가 전달이 되는데, 람다 안에서 이 exchange를 capture로 "가져온다"는 느낌이다.
이렇게 하면 exchange의 변경 값을 밖에서 확인할 수 있게 된다.

Atomic이라는 이름이 붙은 것처럼 멀티스레드 환경에서 안전하게 값을 교체하려고 쓰는 건데, 사실 테스트에서는 람다 밖으로 값 빼기 편해서 사용하는 경우가 많다.
### Mockito로 만들면 “진짜 구현”이 아니라 행동을 흉내내는 가짜 객체가 된다.
실제 로직이 실행되지 않음(부작용 없음)
메서드 호출 여부/횟수/인자를 검증 가능
필요하면 원하는 반환값을 미리 지정(stubbing)
즉, 복잡한 의존성 없이 **상호작용만 검증**하고 싶을 때 쓴다.
### @Mock
@Mock으로 생성한 객체들은 실제로 생성된 것이 아닌 이름만 가져온 것이다.
즉, 해당 객체가 가지고 있는 메서드를 사용하려면 
`when(userRepository.findByEmail(request.getEmail())).thenReturn(Optional.of(user));`
이렇게 메서드 호출 시 지정해둔 리턴값을 반환하도록 미리 설정해두어야 한다.
### @InjectMocks
Mock으로 생성된 객체들을 해당 객체에 넣을 수 있다.
이로 인해 빈으로 생성되지 않은 객체들도 생성된 것처럼 해당 객체에서 사용할 수 있게 된다.

## https://margin1103.tistory.com/61 정리본
통합 테스트를 진짜 인프라로 돌리고 싶은데 환경 관리가 너무 번거롭다.
Testcontainers로 테스트에서 자동으로 띄워서 해결한다.
### TestContainers가 뭐냐면..
도커 컨테이너로 실행되는 실제 서비스를 테스트에서 자동으로 띄우고 내리는 라이브러리를 의미한다.
Mock이나 인메모리 대체재 없이, 운영과 같은 종류의 서비스와 통신하는 통합 테스트를 쉽게 만들어준다.
### 왜 필요해?
테스트 전에 인프라가 켜져 있는지, 데이터가 준비됐는지 매번 확인하고 관리해주어야 하며, CI에서 여러 파이프라인이 병렬로 돌 때 데이터를 오염시키거나 간섭이 생겨 테스트가 불안정해질 수 있다.
운영 DB 대신 사용하는 대체재가 기능을 전부 지원하지 못해 문제가 생길 수도 있고, 테스트에서는 되는데 운영에서는 실패하는 문제가 발생할 수도 있다.

