# AuthenticationManager와 Authentication

AuthenticationManager에서 인증을 처리하는 과정을 살펴보자.  

## AuthenticationManager

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

AuthenticationManager는 authenticate 메서드 하나 뿐이다. 

<br>

**authenticate**

 * 인자로 받은 authentication이 인증 정보를 담고 있다. (유저가 입력한 username과 password)
 * 인증 정보가 유효한지 확인하고 유효하다면 authentication 객체를 반환한다. (UserDetailsService가 리턴한 그 객체를 principal로 담고있는 authentication)
 * 예외 종류
    - `badCredentialsException` : 비밀번호 오류
    - `LockedException` : 계정이 잠겨있는 경우
    - `DisableException` : 계정이 비활성화된 경우


<br>

<br>  

## ProviderManager

AuthenticationManager의 구현체로는 보통 ProviderManager를 사용(직접 구현도 가능)

Debugger를 이용해 어떻게 이루어져 있는지 확인해보자.

**1.** `Class<? extends Authentication> toTest = authentication.getClass();`
![image](https://user-images.githubusercontent.com/57824259/233538825-1ecb960f-c952-4df7-b90f-a321db8e0a45.png)

* 입력한 정보는 두개밖에 없어서 아직 들어 있는게 없다.

<br>  

**2. provideManager가 직접하는 것이 아니라 또 다른 provider들을 사용하여 인증한다.**
```java
ProviderManager의 authenticate 메서드

     ...

Iterator var9 = this.getProviders().iterator();

while(var9.hasNext()) {
    AuthenticationProvider provider = (AuthenticationProvider)var9.next();
    if (provider.supports(toTest)) {
        if (logger.isTraceEnabled()) {
            Log var10000 = logger;
            String var10002 = provider.getClass().getSimpleName();
            ++currentPosition;
            var10000.trace(LogMessage.format("Authenticating request with %s (%d/%d)", var10002, currentPosition, size));
        }
        
    ...
    
    
```
<br>  

**3. ProviderManager가 parent가 있다.**

![image](https://user-images.githubusercontent.com/57824259/233540143-c8353656-cffd-4595-91f5-08a0752020da.png)
* ProviderManager는 AuthenticationProvider로 딱 하나 가지고 있는데 그 provider가 익명 사용자를 인증하는 AnonymousAuthenticationProvider이다.
우리가 폼으로 넘겨준 authentication객체의 실제 타입인 UsernamePasswordAuthenticationToken으로 인증 하지 못한다.
  -> 이 경우 아래와 같이 parent로 가서 인증을 해달라고 한다.

```java
if (result == null && this.parent != null) {
    try {
        parentResult = this.parent.authenticate(authentication);
        result = parentResult;
    } catch (ProviderNotFoundException var12) {
    } catch (AuthenticationException var13) {
        parentException = var13;
        lastException = var13;
    }
}
```

<br>  

**4. DaoAuthenticationProvider**

![image](https://user-images.githubusercontent.com/57824259/233540426-3b174369-6a88-4063-b2d9-62faa626005a.png)

* 이 Provider는 UsernamePasswordAuthenticationToken 인증을 처리할 수 있다.
*  DaoAuthenticationProvider의 supports 
![support](https://user-images.githubusercontent.com/57824259/233540610-fe1bc6c5-39a9-4854-ab73-1b0d798a6969.PNG)

<br>  

**5. abstractUserDetailsAuthenticationProvider**

* 이름에서 유추할 수 있듯이 UserDetailService를 사용해서 인증을 해주는 Provider이다.
* `retrieveUser`는 DaoAuthenticationProvider의 retrieveUser으로 아래에서 확인할 수 있듯이 UserDetailsService와 연결을 시켜준다.
![daoAuthenticationProvider 유저디테일서비스 연결](https://user-images.githubusercontent.com/57824259/233540916-b8e9028b-4ae6-41f4-b970-b19a2a185c77.PNG)
* `UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);`  : UserDetailsService가 우리가 만든 그 UserDetailsService인 AccountService이다.
* AccountService에서 Username을 통해 User를 가져오는 것

```java
AccountService

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    Account account = accountRepository.findByUsername(username);
    if(account == null){
        throw new UsernameNotFoundException(username);
    }

    return User.builder()
            .username(account.getUsername())
            .password(account.getPassword())
            .roles(account.getRole())
            .build();
}
```
* `UserDetails` : 어플리케이션만의 account와 principal과의 어뎁터 역할을 한다.

<br>  

**6. 이후 추가적인 인증 (ex. 계정이 잠겨있는지 등..)**

<br>  

**7. 캐시**

<br>  

**8. authentication을 가져와 result를 리턴하게 된다.**
  ![image](https://user-images.githubusercontent.com/57824259/233542915-febf31d4-3749-4251-92a7-dc3a38fda1f0.png)
  
  같은 UsernamePasswordAuthenticationToken인데 principal이 다르다.
  
  문자열 -> loadByUsername에서 반환한 User 객체를 들고있다.
  
<br>

**9.이 authentication 객체가 SecurityContext에 담기는 것이다.**
