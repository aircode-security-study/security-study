# 스프링 시큐리티 필터와 FilterChainProxy

스프링 시큐리티가 제공하는 필터들

* WebAsyncManagerIntergrationFilter
* **SecurityContextPersistenceFilter**
* HeaderWriterFilter
* CsrfFilter
* LogoutFilter
* **UsernamePasswordAuthenticationFilter**
* DefaultLoginPageGeneratingFilter
* DefaultLogoutPageGeneratingFilter
* BasicAuthenticationFilter
* RequestCacheAwareFtiler
* SecurityContextHolderAwareReqeustFilter
* AnonymouseAuthenticationFilter
* SessionManagementFilter
* ExeptionTranslationFilter
* FilterSecurityInterceptor

<br>

## FilterChainProxy

FilterChainProxy의 `doFilterInternal` 내부의 `getFilters`로 필터 목록을 가져오고 chain을 가지고 VirtualFilterChain을 하면

`doFilter`를 한다.

<br>

## FilterChainProxy와 SecurityConfig

1. FilterChainProxy가 chain의 목록을 가져온다. : `getFilters` - 특정한 요청이 매치가 되면 chain의 필터 목록을 가져온다
2. FilterChain의 목록은 SecurityConfig 클래스에서 생성된다.
3. SecurityConfig 자체가 하나의 FilterChain이라고 생각을 해도 된다. -> 여러개 만들 수 있다.
4. 여러개 필터 체인 중 매치가 되는 것을 사용한다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    AccountService accountService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/","/info", "/account/**").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated();
        http.formLogin();
        http.httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(accountService);
    }
}
```

#### `antMatcher`란?

* `antMatcher` 시큐리티 필터 체인 안에서 매치를 할때 사용하는 정보이다.
* 아무것도 안주면 모든 요청을 다 매핑하는 것이 된다.

<br>

## SecurityConfig가 중복된 경우

SecurityConfig이 상충되는 경우 스프링 시큐리티가 미연에 방지를 해두었다.

```java
// AnotherSecurityConfig

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .anyRequest().authenticated();
    http.formLogin();
    http.httpBasic();
}
    
// --------------- 

// SecurityConfig
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .anyRequest().permitAll();
    http.formLogin();
    http.httpBasic();
}
    
```

위와 같이 중복될 경우 다음과 같은 에러가 발생한다.

![상충될 경우](https://user-images.githubusercontent.com/57824259/233794332-1370de4a-965c-4929-9af0-0f225ab90c98.PNG)

## 해결 방법

* `@Order(Ordered.LOWEST_PRECEDENCE - 15)` 어노테이션 추가 (우선순위를 설정해준다)

![22222](https://user-images.githubusercontent.com/57824259/233794568-0d38bd9d-914a-4cf5-9b47-d3e7dffb2bd7.PNG)


## 정리

* Filter들의 목록은 설정에 따라 달라진다. (추가, 커스터마이징)
* FilterChainProxy가 여러개의 Filter들을 관리하고 실행하는 것을 확인
* FilterChainProxy가 Filter들을 순서대로 쭉 호출을 해준다.
* antMatcher에 따라 Order 우선순위에 따라 선택이 되어 실행을 한다.

<br>

## 다음 내용

* 어떻게 FilterChainProxy까지 들어왔는지 살펴본다.
