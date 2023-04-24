# DelegatingFilterProxy와 FilterChainProxy

## DelegatingFilterProxy 살펴보기

1. 어떠한 요청을 보내면 서블릿 컨테이너(Tomcaat)가 받는다.
2. 서블릿 스펙에는 서블릿 필터라는 개념이 있다.
3. 서블릿 필터란, 요청을 처리하는 앞 뒤로 특정한 일들을 할 수 있는 인터셉터 역할이다.
4. 구현체 중 하나가 `DelegatingFilterProxy`이다.
5. `DelegatingFilterProxy`란, 다른 객체에게 위임을 하는 프록시이다.
6. `DelegatingFilterProxy`는 스프링 ioc 컨테이너에 있는 빈에게 해야할 일을 위임을 한다. (`SpringSecurityFilterChain`에게 위임한다고 볼 수 있다.)
7. 어떤 빈에게 위임을 할지 명시를 해줘야 한다.
    * Spring boot가 아닌 경우 `AbstractSecurityWebApplicationinitializer`를 쓰면 자동으로 등록
    * Spring boot를 쓸때는 빈(`SecurityFilterAutoConfiguration`)설정에 따라서 자동으로 등록
    * 여기서, `SecurityFilter`이란 서블릿 필터이다.(사실 모든 필터가 서블릿 필터이다.)

8. DelegatingFilterProxy는 등록되는 위치와 사용되는 방법이 다르다.
9. DelegatingFilterProxy를 제외한 나머지는 Spring 내부적으로 사용되는 필터들인 것이고 인터페이스만 서블릿 필터이다.
10. 작업을 위임하려면 빈의 이름을 알아야 한다. = `SpringSecurityFilterChain`

<br>

## 정리

* SecurityContextHolder -> Authentication을 가지고 있고 AuthenticateManager가 인증 -> SecurityContextHolder에 저장
* 여러가지 Filter들은 FilterChainProxy가 호출을 해주고 FilterChainProxy로는 DelegatingFilterProxy에 의해 들어왔다.
* 사실은 Order 어노테이션보다 `antMatcher("/**")` 로 매칭하는걸로 필터체인을 설정해야 한다.

<br>

## 다음 내용

* 권한 확인이 어디서 이루어지는지 살펴보기.
