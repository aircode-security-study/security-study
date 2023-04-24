# AccessDecisionManager

Access Control(또는 Authorization)을 다루는 인터페이스를 중심으로 살펴보자.

<br>

## AccessDecisionManager의 역할

이미 인증을 거친 사용자가 특정한 서버 리소스에 접속을 할때 허용할 것인가? 유효한 요청인가 판단하는 역할이다.

AuthtenticationManager : 인증(Authentication)
AccessDecisionManager : 인가(Authorization)

<br>

## AccessDecisionManager의 구현체

* **AffirmativeBased** : 여러 Voter 중에 한명이라도 허용하면 허용. 기본 전략.
* ConsensusBased : 다수결
* UnanimousBased : 만장일치

AccessDecisionManager은 여러개의 Voter를 가질 수 있다.  
의사 결정을 내릴때 Access Control 안에 들어있는 Voter들을 거치면서 유효한지 판단을 한다.  
해당한 전략에 따라 허용하지 않으면 예외를 발생한다.  

```java

public interface AccessDecisionManager {
    void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException, InsufficientAuthenticationException;

    boolean supports(ConfigAttribute attribute);

    boolean supports(Class<?> clazz);
}

```

`decide` : decide 메서드를 반드시 구현해야 한다.  
`supports` : ConfigAttribute를 지원하는지 판단하는 메서드  
`ConfigAttribute` : 예시로 permitAll(), hasRole("ADMIN")  

```java
public class AffirmativeBased extends AbstractAccessDecisionManager {
    public AffirmativeBased(List<AccessDecisionVoter<?>> decisionVoters) {
        super(decisionVoters);
    }

    public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
        int deny = 0;
        Iterator var5 = this.getDecisionVoters().iterator();

        while(var5.hasNext()) {
            AccessDecisionVoter voter = (AccessDecisionVoter)var5.next();
            int result = voter.vote(authentication, object, configAttributes); // * 여기에서 vote
            switch(result) {
            case -1:
                ++deny;
                break;
            case 1:
                return;
            }
        }

        if (deny > 0) {
            throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        } else {
            this.checkAllowIfAllAbstainDecisions();
        }
    }
}
```

**result값**

* 1 허용 (ACCESS_GRANTED)
* 0 애매 (ACCESS_ABSTAIN)
* -1 거부 (ACCESS_DENIED)

허용이 되면 경로에 접근이 된다.

<br>

## AccessDecisionManager 또는 Voter를 커스터마이징 하는 방법

AccessDecisionVoter가 하나밖에 없는데 어떻게 커스터마이징이 가능한지 살펴보자

```java
@GetMapping("/user")
public String user(Model model, Principal principal){
    model.addAttribute("message", "Hello User, " + principal.getName());
    return "user";
}
```

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/","/info", "/account/**").permitAll()
            .mvcMatchers("/admin").hasRole("ADMIN")
            .mvcMatchers("/user").hasRole("USER")
            .anyRequest().authenticated();
    http.formLogin();
    http.httpBasic();
}
```

다음과 같이 설정하고 ADMIN 역할을 가진 사용자로 /user에 접근하면 403 Forbidden이 나온다.

![image](https://user-images.githubusercontent.com/57824259/233837243-0461ac66-c99b-47a4-b134-8b619a53938a.png)

스프링 시큐리티는 ADMIN이 무슨 뜻인지 알 수 없다.

<br>

**커스터 마이징 방법 1**

 ADMIN을 반환할때 USER를 같이준다.
 
```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    Account account = accountRepository.findByUsername(username);
    if(account == null){
        throw new UsernameNotFoundException(username);
    }

    return User.builder()
            .username(account.getUsername())
            .password(account.getPassword())
            .roles(account.getRole() , "USER")
            .build();
}
```

**커스터 마이징 방법 2**

AccessDecisionVoter 또는 AccessDecisionManager가 ROLE(hierarchy)을 이해하는 방법을 사용한다.

```java

// 커스터마이징
public AccessDecisionManager accessDecisionManager(){
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");

    DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
    handler.setRoleHierarchy(roleHierarchy);

    WebExpressionVoter webExpressionVoter = new WebExpressionVoter();
    webExpressionVoter.setExpressionHandler(handler);

    List<AccessDecisionVoter<? extends Object>> voters = Arrays.asList(webExpressionVoter);

    return new AffirmativeBased(voters);
}
    
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/","/info", "/account/**").permitAll()
            .mvcMatchers("/admin").hasRole("ADMIN")
            .mvcMatchers("/user").hasRole("USER")
            .anyRequest().authenticated()
            .accessDecisionManager(accessDecisionManager()) // 이렇게 명시를 하지 않으면 AffirmativeBased 사용
    ;
    http.formLogin();
    http.httpBasic();
}
```

hierarchy를 설정할 수 있는 것은 핸들러이다.

`roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER")` : ROLE_ADMIN은 ROLE_USER의 상위 버전이다.

**커스터 마이징 방법 3**

accessDecisionManager를 커스터마이징을 하는 것이 아니라 SecurityExpressionHandler만 커스터마이징 하는 것이다.  

```java
// SecurityExpressionHandler 커스터마이징
public SecurityExpressionHandler expressionHandler(){
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");

    DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
    handler.setRoleHierarchy(roleHierarchy);

    return handler;
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/","/info", "/account/**").permitAll()
            .mvcMatchers("/admin").hasRole("ADMIN")
            .mvcMatchers("/user").hasRole("USER")
            .anyRequest().authenticated()
            .expressionHandler(expressionHandler())
    ;
    http.formLogin();
    http.httpBasic();
}

```

그나마 소스의 길이을 줄이기 위해 3번째 방법을 권유한다.

**다음 내용**

* AccessDecisionManager는 누가 사용하는가?
