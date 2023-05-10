# Spring Security 커스터 마이징

자동 설정은 UserDetailsServiceAutoConfiguration에서 해준다.

```java
public class UserDetailsServiceAutoConfiguration {
    private static final String NOOP_PASSWORD_PREFIX = "{noop}";
    private static final Pattern PASSWORD_ALGORITHM_PATTERN = Pattern.compile("^\\{.+}.*$");
    private static final Log logger = LogFactory.getLog(UserDetailsServiceAutoConfiguration.class);

    ...

    @Bean
    @Lazy
    public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties, ObjectProvider<PasswordEncoder> passwordEncoder) {
        User user = properties.getUser();
        List<String> roles = user.getRoles();
        return new InMemoryUserDetailsManager(new UserDetails[]{org.springframework.security.core.userdetails.User.withUsername(user.getName()).password(this.getOrDeducePassword(user, (PasswordEncoder)passwordEncoder.getIfAvailable())).roles(StringUtils.toStringArray(roles)).build()});
    }

    ...
    
}
```

#### 인메모리 방식

```java
// SecurityConfig.class

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            .withUser("wooseong").password("{noop}123").roles("USER").and()
            .withUser("admin").password("{noop}!@#").roles("ADMIN");
            
            // 여기서 {noop}이란 : 암호화를 하지 않음, security 5.부터 사용하는 기본 password encoder
}
```

위의 소스와 같이 인메모리에 유저정보를 여러개, 원하는 유저정보를 임의로 설정 가능하다.

<details>
<summary> 내용 보기 </summary>
<div markdown="1">

auth.inMemoryAuthentication()를 실행하게 되면 InMemoryUserDetailsManagerConfigurer(UserDetailsManagerConfigurer 상속)가 반환되는데
withUser 메서드를 통해 원하는 유저정보를 추가한다.

</div>
</details>

#### JPA 연동 방식


```java
// SecurityConfig.class

@Autowired
AccountService accountService;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(accountService);
}
```

명시적 선언 : auth.userDetailsService에 userDetails를 반환해줄 수 있는 서비스(예시.AccountService)를 주입
Bean으로 등록하게 되면 가져다 쓴다.

```java
// AccountService.class

@Service
public class AccountService implements UserDetailsService {
    // data access object 인터페이스를 통해서 읽어오는 UserDetailsService 인터페이스

    // 단일책임원칙에 의해 하나의 역할만 있어야 객체지향적이긴 하지만
    // 이 일도 accountService의 일부라 생각하기 때문에 loadUserByUsername를 여기에 추가
    // TODO {noop}123
    @Autowired
    AccountRepository accountRepository; // 이자리에는 임의의 dao 구현체가 와도 된다. nosql 구현체, 관계형 db 구현체

    @Autowired
    PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = accountRepository.findByUsername(username);
        if(account == null){
            throw new UsernameNotFoundException(username);
        }

        // User가 UserDetail을 주입
        return User.builder()
                .username(account.getUsername())
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }

    ...
    
}
```

**{noop} 없애는 방법**  

```java
// spring security 5 이전 버전의 인코더

@SpringBootApplication
public class DemoSpringSecurityFormApplication {

	@Bean
	public PasswordEncoder passwordEncoder(){
		return NoOpPasswordEncoder.getInstance();
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoSpringSecurityFormApplication.class, args);
	}

}
```

5이후는 {id}encodedPassword 포맷이 왜 생겼느냐?

기본 전략을 noop -> bcrypt 전략으로 바꿨다.

버전을 올리면 인증이 깨진다. 또는 다른 알고리즘 전략을 쓰고 싶기도 하다.

 -> 여러가지 패스워드인코딩을 지원하려다 보니 이러한 기본 포맷을 선택하게 됨


```java
// Spring security 5 이후

@SpringBootApplication
public class DemoSpringSecurityFormApplication {

	@Bean
	public PasswordEncoder passwordEncoder(){
		return PasswordEncoderFactories.createDelegatingPasswordEncoder();
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoSpringSecurityFormApplication.class, args);
	}

}
```



# Spring Security 테스트 코드 작성

#### mockMvc.perform를 사용하여 가로채 해당 유저로 이미 로그인 한 상태로 만듬

**테스트 유저 설정 방법**

1 : mockMvc.perform의 with을 이용한 방법

```java
@Test
public void admin_user() throws Exception{
    mockMvc.perform(get("/admin").with(user("wooseong").roles("USER")))
            .andDo(print())
            .andExpect(status().isForbidden());
}
```

2 : @WithMockUser 어노테이션 추가

```java
@Test
@WithMockUser(username = "wooseong", roles = "USER") // .with(user("wooseong").roles("USER")))가 필요없다
public void index_user() throws Exception{
    mockMvc.perform(get("/"))
            .andDo(print())
            .andExpect(status().isOk());
}
```


3 : 메타 어노테이션 생성 @WithUser

```java
@Test
@WithUser
public void admin_user() throws Exception{
    mockMvc.perform(get("/admin").with(user("wooseong").roles("USER")))
            .andDo(print())
            .andExpect(status().isForbidden());
}
```

  WithMockUser 어노테이션(2번)을 가진 새로운 어노테이션으로 정리

```java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "wooseong", roles = "USER")
public @interface WithUser {
}
```

**테스트 코드 작성 시 주의사항**

PasswordEncoder를 Bean으로 등록해서 서비스에서 받아서 인코더하도록 해야 패스워드 일치 테스트간 혼용을 방지(인코더 된 패스워드랑 비교 x)
