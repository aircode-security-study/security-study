# Spring Security 연동하기

#### 프로젝트 버전

- `Java` : 11
- `Spring-boot` : 2.7.11
- `Spring-security` : 5.7.8
- `H2 database` : 2.1.214

#### 의존성 추가하기(Maven)

스프링 부트의 도움을 받아 dependency에서 spring-boot-starter-security 추가

```html
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  
  <!--  Spring security test  -->
  <dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
    <version>${spring-security.version}</version>
  </dependency>
</dependencies>
```

위와 같이 의존성만 추가할 경우 모든 요청은 인증을 필요로 하고 하나의 기본 유저가 생성된다.

의존성만 추가하면 쓸 수 있는가? No  
고정된 유저 정보와 로그에 남기도 하며 변하는 패스워드이므로 커스터마이징이 필요.  

<br>
  
# Spring Security 설정하기

WebSecurityConfigurerAdapter를 통해 configure 메서드를 Override 하여 설정한다.

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
        http.formLogin(); // logout 기능도 지원
        http.httpBasic();
    }
}
```

@EnableWebSecurity 없어도 스프링 부트에서 자동 설정

이때, **WebSecurityConfigurerAdapter**는 Spring Security 5.7.0-M2에서는 비권장한다.  
[자세한 내용 보기](https://github.com/spring-projects/spring-security/issues/10822)  

5.7 이후 버전부터는 **SecurityFilterChain**을 bean으로 주입해서 사용한다.  
[관련 문서](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authz) -> authz
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults())
            .authenticationManager(new CustomAuthenticationManager());
        return http.build();
    }

}
```

**HttpSecurity의 메서드**

- authorizeHttpRequests : RequestMatcher 구현(예: URL 패턴을 통해)을 사용하여 HttpServletRequest를 기반으로 액세스를 제한
- mvcMatchers : 경로 패턴 추가
- permitAll, hasRole : 권한 설정
- anyRequest : 나머지 경로
- authenticated : 인증된 사용자만 접근
- formLogin : form login, logout 기능 사용
- httpBasic : httpBasic 설정 (자세한 내용은 뒤에서 다룸)
