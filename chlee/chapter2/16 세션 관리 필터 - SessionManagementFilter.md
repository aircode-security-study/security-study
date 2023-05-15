# **SessionManagementFilter 란?**

스프링 시큐리티 내부에서 중요한 역할을 하는 필터이며, 몇가지 중요한 기능들을 제공해줍니다.

1. 세션 변조 방지 전략 설정: sessionFixation
2. 유효하지 않은 세션을 리다이렉트 시킬 URL 설정					
3. 동시성 제어: maximumSessions					
4. 세션 생성 전략: sessionCreationPolicy

<br>

# 세션 변조 방지 전략 설정: sessionFixation

## 세션 변조?

![Untitled](https://github.com/aircode-security-study/security-study/assets/77683221/43692b03-941a-4d54-aa1d-d11ec212e10a)


1. 어떤 공격자가 어떤 웹 사이트에 로그인합니다.
2. 공격자 자신의 sessionid를 쿠키로 받아옵니다.
3. 그리고 쿠키에 들어있는 sessionid를 희생자에게 보냅니다.
4. 희생자는 받아진 sessionid를 가지고 웹 사이트에 접근해서 로그인을 하게 되면 웹 사이트의 입장에서는 공격자와 희생자를 같은 사람으로 인식합니다.
    
    그래서 공격자가 희생자의 개인정보를 마음대로 볼 수 있게 됩니다.
    

## 세션 변조 방지

이런 것을 방지하는 가장 간단한 방법이 인증을 했을 때 세션을 새로 만들거나 sessionid 를 바꾸는 것입니다.

인증을 했을 때 세션을 바꾸게 되면 쿠키에 들어있는 sessionid가 공격자의 sessionid와 다르기 때문에 희생자의 정보를 볼 수 없습니다.

그래서 스프링 시큐리티에는 세션 변조 방지 전략이 있습니다. 세션 변조 방지 전략은 서블릿 컨테이너에 따라 달라집니다.

- migrateSession (서블릿 3.0- 컨테이너 사용시 기본값)
    - 서블릿 3.0 이하부터는 migrateSession 전략을 사용합니다.
    - 인증이 되었을 때 새로운 세션을 만들고 기존에 있는 세션의 attribute를 새로운 세션에 복사해옵니다.
- changeSessionId (서블릿 3.1+ 컨테이너 사용시 기본값)
    - 세션 id 를 바꾸는 방법은 서블릿 3.1 부터 지원합니다.
    - changeSessionId 는  migrateSession 보다 좀 더 좋습니다. migrateSession는 세션을 새로 만들고 복사를 해야하는데 changeSessionId 는 sessionid 만 만들기 때문에 성능이 좋겠죠.

> 참고로 저의 톰캣 버전은 9.0.58 이고 톰캣 버전이 9.x 라면 서블릿 스펙 4.0을 지원합니다.
> 
> 
> <img width="497" alt="Untitled 1" src="https://github.com/aircode-security-study/security-study/assets/77683221/59647161-11d4-4929-bf4a-be89a55eea26">
> 
> 그러면 changeSessionId 전략을 사용하고 있다고 볼 수 있습니다.
> 

## 세션 변조 방지 전략 설정

- SecurityConfig

```java
@Override
protected void configure(HttpSecurity http) throws Exception {

		...

		http.sessionManagement()
					.sessionFixation() //...1
					. ...

		...
}
```

1. 세션 변조 방지를 설정할 수 있습니다.
    - 4가지 전략을 지원합니다.
    
    <img width="601" alt="Untitled 2" src="https://github.com/aircode-security-study/security-study/assets/77683221/db081bda-5b96-46c8-a7c0-626908ebc121">

    
    1. none() : 세션 변조를 신경쓰지 않겠다.
    2. newSession() : 새로운 세션을 만들겠다.
        - 기존 세션의 attribute를 가져오지 않는다는 의미라서 사용할 이유가 없습니다.
    3. migrateSession()
    4. changeSessionId()

사실 딱히 설정할 이유가 없습니다.

# 유효하지 않은 세션을 리다이렉트 시킬 URL 설정 : invalidSessionUrl

유효하지 않은 세션은 로그아웃 했을 때 일어납니다.

로그아웃 했을 때 기존에 있던 세션을 invaild 시킵니다. 더 이상 유효하지 않은 세션으로 간주하죠.

- SecurityConfig

```java
@Override
protected void configure(HttpSecurity http) throws Exception {

		...

		http.sessionManagement()
					.invalidSessionUrl("/error"); //..1

		...
}
```

1. 유효하지 않은 세션이 접근했을 때 어디로 보낼지 정할 수 있습니다. 현재는 error 페이지로 보내도록 설정했습니다.
    - 보통 error 페이지로 보내거나 login 페이지로 보내는게 좋겠죠.

# 동시성 제어: maximumSessions

한 유저의 계정이 동시에 로그인 될 수 있습니다. 스프링 기본값은 동시 로그인이 가능하도록 설정되어 있죠.

그런데 하나의 계정에 대해서 동시에 딱 하나의 세션만 유지하고 싶을 때는 maximumSessions 를 설정할 수 있습니다.

- SecurityConfig

```java
@Override
protected void configure(HttpSecurity http) throws Exception {

		...

		http.sessionManagement()
						.maximumSessions(1) //...1
								.maxSessionsPreventsLogin(true); //...2-b

		...
}
```

1. 하나의 계정에 하나의 세션만 유지하도록 설정합니다.
2. 부가적인 설정
    1. expireUrl()
        - maximumSessions을 1로 설정했는데 여러개의 세션이 로그인을 시도할 때 기본전략은 이전 세션을 만료시키는 것입니다.
            
            예를 들어, 크롬으로 로그인을 했을 때는 첫 번째니까 로그인이 되고 그 다음에 파이어폭스로 로그인을 하면 로그인이 되지만 크롬에서 세션이 만료됩니다.
            
            이 때 만료되었을 때 어디로 보낼지 설정할 수 있는게 바로 expireUrl 입니다.
            
    2. maxSessionsPreventsLogin()
        - 기존의 세션을 만료시키는게 아니라 새로운 세션으로 로그인을 못하게 하고 싶을 때 maxSessionsPreventsLogin 를 true로 설정하면 됩니다.
        - 기본값은 false입니다.
            
            ![Untitled 3](https://github.com/aircode-security-study/security-study/assets/77683221/799aa7f9-8978-446b-be11-6b66c9ad6edf)

            
            다른 곳에서 로그인을 한다면 기존의 세션을 만료시키고 로그인이 됩니다.
            

# 세션 생성 전략: sessionCreationPolicy

- SecurityConfig

```java
@Override
protected void configure(HttpSecurity http) throws Exception {

		...

		http.sessionManagement()
						.sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED); //...1

		...
}
```

1. 세션 생성 전략을 설정합니다.
    1. IF_REQUIRED
        - 기본값 입니다.
        - 필요하면 세션을 만듦니다.
    2. NEVER
        - 스프링 시큐리티에서 만들지 않고, 기존에 세션이 있다면 세션을 사용합니다.
    3. STATELESS
        - 세션을 아예 사용하지 않습니다.
        - NEVER 는 기존에 세션이 있다면 사용하는 것이지만 STATELESS 는 진짜 사용하지 않는 것입니다.
        - STATELESS를 사용하게 되면 인증이 필요한 요청이라면 계속 로그인을 해야합니다.
    4. ALWAYS
        - 항상 세션을 생성합니다.

<aside>
❗ 세션 생성 전략을 STATELESS 로 설정했을 때 로그인 후에 무조건 / 페이지로 가는 이유?

---

기존에 /dashboard 요청을 보내면 로그인 페이지가 나오고 로그인을 성공하면 /dashboard 페이지가 나왔습니다.

하지만 세션 생성 전략을 STATELESS 로 설정하게 되면 로그인에 성공할 경우에 /dashboard 페이지가 아니라 / 페이지로 이동하게 됩니다.

그 이유는 RequestCacheAwareFilter 때문입니다.

RequestCacheAwareFilter에서 캐시를 세션에 저장하기 때문이죠.

즉, 세션을 안쓰니까 캐시를 못찾아서 / 페이지로 가는 것입니다.

그래서 폼 인증 기반의 에플리케이션이라면 세션을 사용하는게 낫습니다.

만일 서버를 여러개 둬야해서 여러개의 서버에서 세션을 공유하는게 걱정이라면 spring session 을 사용해서 세션 클러스터를 좀 더 쉽게 구성할 수 있습니다.

→ [spring session 공식문서](https://docs.spring.io/spring-session/reference/index.html)

</aside>

<aside>
❗ 세션 클러스터링

---

세션 클러스터링은 분산되어 있는 다수의 WAS Instance를, 세션을 공유하는 하나의 논리적 그룹으로 묶어 부하분산 및 네트워크, 서버의 장애 시 사용자의 접속 정보 유실을 방지하고 안정적인 서비스를 제공하기 위해 필요한 기능입니다.

</aside>

# Reference

- 세션 변조
    - [https://owasp.org/www-community/attacks/Session_fixation](https://owasp.org/www-community/attacks/Session_fixation)
- 세션 클러스터링
    - [http://controlcenter.kr/html/clustering01.html](http://controlcenter.kr/html/clustering01.html)