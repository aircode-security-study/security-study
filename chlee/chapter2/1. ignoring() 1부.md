# 설정한 시큐리티 필터

모든 요청은 스프링 시큐리티가 필터들을 적용해서 처리를 했습니다.

실제로 어떤 웹 페이지 요청을 할 때 우리가 설정한 아래의 필터들이 적용되었죠.

- SecurityConfig

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .mvcMatchers("/", "/info", "/account/**").permitAll()
            .mvcMatchers("/admin").hasRole("ADMIN")
            .mvcMatchers("/user").hasRole("USER")
            .anyRequest().authenticated()
            .expressionHandler(expressionHandler());

    http.formLogin();
    http.httpBasic();
}
```

# / 요청을 보냈을 때 벌어지는 일?

[http://localhost:8080/](http://localhost:8080/) 에 요청을 보내면 /에 대한 요청만 갈 것 같지만 실제로는 3가지 요청이 갑니다.

![Untitled](https://user-images.githubusercontent.com/77683221/236993596-3a3fc50f-373f-425f-8056-cecdbd9c8a45.png)

우리는 / 에 대한 요청만 직접적으로 보냈지만 브라우저가 기본적으로 favicon 요청을 보냅니다.

그래서 favicon.ico 라는 요청이 스프링 시큐리티에 걸리는데 설정했던 SecurityConfig 에서 `.anyRequest().authenticated()` 에 걸리게 됩니다. 그러니 인증을 필요로 하는 요청으로 처리가 됩니다.

그래서 login 요청까지 추가로 보내게 되는겁니다.

<br>


하지만 / 에 대한 응답을 받았기 때문에 우리가 원하는 화면을 뿌려주고 있는 것이죠.

여기서 문제는 제대로 동작한 것처럼 보이지만 필요 없는 리소스를 쓰게 되었다는 것 입니다.

<br>

<aside>
❗ 파비콘( favicon )이란?

---

- 아래 그림과 같이 파란동그라미가 그려져 있는 부분이 파비콘입니다.
    
    ![Untitled 1](https://user-images.githubusercontent.com/77683221/236993726-288d7fdb-1e04-4ce3-b36f-ef44c7116c89.png)

    
- 서버에 있는 파비콘이라는 정적 리소스를 브라우저가 자동으로 요청합니다.
</aside>

<br>

## 디버깅

위의 3가지 요청의 동작을 자세히 보기 위해 FilterChainProxy에 디버거를 걸어봅시다.

- FilterChainProxy

<img width="1219" alt="Untitled 2" src="https://user-images.githubusercontent.com/77683221/236993811-7c0b438c-8953-44c3-a94c-dec90b604dac.png">

앞에서 봤듯 FilterChainProxy 에서 어떤 요청을 받으면 요청에 적용해야하는 필터들의 목록을 getFilters 가 가져옵니다.

처음 걸린 요청은 /에 대한 요청입니다. 그리고 필터는 우리가 설정한 15개의 필터가 적용됩니다.

<img width="357" alt="Untitled 3" src="https://user-images.githubusercontent.com/77683221/236993925-4135ec96-aa88-4b32-b2ae-af12604d098d.png">


<img width="428" alt="Untitled 4" src="https://user-images.githubusercontent.com/77683221/236993989-26f3c3b0-34f1-409d-a6b6-8a1fed99e825.png">


그리고 쭉 실행이 되고 / 페이지가 보여지게 됩니다.

그런데 / 에 대한 요청은 이미 응답이 되고 보여졌는데 서버에 /favicon.ico 에 대한 요청이 또 들어옵니다.

마찬가지로 /favicon.ico 요청에 대해 우리가 설정한 필터들이 적용됩니다.

<img width="424" alt="Untitled 5" src="https://user-images.githubusercontent.com/77683221/236994029-27e61498-e55a-4e32-9c8b-96080737b726.png">

<img width="428" alt="Untitled 6" src="https://user-images.githubusercontent.com/77683221/236994059-cde6d2b5-7dde-487c-b7b3-7fb24b3b8930.png">


/favicon.ico 요청은 15개의 필터 중 FilterSecurityInterceptor 에 걸립니다.

인증된 사용자가 없는데 SecurityConfig에서 인증된 사용자만 접근하도록 설정을 했기 때문입니다.

그러니까 FilterSecurityInterceptor 에서 예외가 발생하고 ExceptionTranslationFilter 가 예외를 잡아서 로그인을 처리하는 AuthenticationEntryPoint 를 실행해서 로그인 페이지로 이동합니다. (정확히는 /favicon.ico 요청을 로그인 페이지로 리다이렉트를 시킵니다.)

그래서 로그인 요청이 한 번 더 오게 됩니다. 역시 마찬가지로 우리가 설정한 필터들이 적용됩니다.

<img width="384" alt="Untitled 7" src="https://user-images.githubusercontent.com/77683221/236994152-2fd0c103-93b2-4295-b482-1f53e262e17c.png">


<img width="422" alt="Untitled 8" src="https://user-images.githubusercontent.com/77683221/236994175-9dbce660-9203-40fb-b0c9-4ddd69c07d47.png">


이 때 DefaultLoginPageGeneratingFilter 에 걸려서 나머지 필터는 타지 않고 응답을 내보내게 됩니다.

로그인 페이지 응답을 브라우저에 돌려주긴 하는데 파비콘에 대한 요청이기 때문에 딱히 보여주지는 않습니다.

# ignoring

그래서 이번에 하고 싶은게 뭐냐면 static resource 를 요청하는 것은 스프링 시큐리티 필터들을 적용하고 싶지 않은 것입니다.

이럴 때는 WebSecurity 를 설정하면 됩니다.

WebSecurity 의 ignoring()을 사용해서 시큐리티 필터 적용을 제외할 요청을 설정할 수 있습니다.

## 방법 1

- SecurityConfig

```java
@Override
public void configure(WebSecurity web) throws Exception { //...1
    web.ignoring().mvcMatchers("/favicon.ico");
}
```

1. WebSecurity 를 파라미터로 받는 메소드를 오버라이딩 합니다.

## 방법 2

문제는 매번 static resource 에 대한 값을 적어주는게 귀찮습니다.

그래서 스프링부트가 제공하는게 있습니다.

- SecurityConfig

```java
@Override
public void configure(WebSecurity web) throws Exception {
		//...1
    web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
}
```

1. 스프링 부트가 제공하는 PathRequest 를 사용해서 정적 자원 요청을 스프링 시큐리티 필터를 적용하지 않도록 설정
    
    스프링 부트가 제공하는 **static** 리소스들의 기본위치를 다 가져와서 스프링 시큐리티에서 제외합니다.
    
    - 정적 리소스의 위치는 Enum으로 구성되어 있습니다. ( StaticResourceLocation )

두 가지 방법 모두 실행을 해서 보면 요청도 2개만 오고 파비콘 요청도 제대로 처리가 됩니다.

파비콘이 필터를 탄 경우에는 인증 처리가 안되서 로그인 폼까지 응답이 가는데 이부분이 생략되니 아까보다 좀 더 빨라집니다. 당연히 리소스와 시간을 쓰는 과정이 생략되니 애플리케이션이 좀 더 빨라지겠죠.

## 디버깅

먼저 / 에 요청을 하면 15개의 필터가 걸리는게 맞습니다.

그 후 /favicon.ico 에 대한 요청이 오면 FilterChainProxy 안에서 파비콘 요청을 처리할 필터들을 가져와야 하는데 이 때 필터들의 목록은 비어있습니다.

<img width="420" alt="Untitled 9" src="https://user-images.githubusercontent.com/77683221/236994343-32320605-3f31-49fd-a4cf-d187d6d47378.png">

<img width="272" alt="Untitled 10" src="https://user-images.githubusercontent.com/77683221/236994370-16e4f98b-d21f-4730-8e3a-5f30c77e0b02.png">

시큐리티 필터를 적용한게 없으니까 바로 스프링 MVC에서 static resource 를 처리하는 것이죠.

파비콘을 따로 설정하지 않았기 때문에 스프링 부트의 기본 favicon 인 나뭇잎 모양이 응답으로 나오게 됩니다.

<br>

<aside>
❗ 스프링 부트가 최근에는 기본 favicon.ico 제공을 하지 않습니다.

그렇기 때문에 ico 파일이 없다면 200 응답이 아니라 404 에러가 발생합니다. 그렇기에 /error 요청이 들어오고 다시 /login 요청까지 발생하죠.

원하는 결과를 보기 위해서는 favicon.ico 파일을 원하시면 다음 경로에 두시면 됩니다.

- resources/static/favicon.ico
</aside>

<br>
<br>

<aside>
❗ ignoring 을 설정할 때 다양한 설정 방법이 존재하며, 더 구체적인 설정도 가능합니다.

---

1. web.ignoring() 에서 제외 방법으로 아래와 같은 방법을 제공합니다.
    - mvcMatchers
    - antMatchers
    - regexMatchers
    - requestMatchers
        - antMatchers, mvcMatchers, regexMatchers 는 결국에 requestMatchers 로 이루어져 있습니다.
        - 스프링부트 3 버전부터는 antMatchers 대신 requestMatchers 을 써야합니다.
2. 일반적으로 사용되는 경로를 만드는데 사용하기 위한 PathRequest 가 제공됩니다.
    - toStaticResources : 정적 리소스를 반환합니다.
        - atCommonLocations : 모든 정적 리소스를 반환합니다.
            - excluding : 필요한 경우 특정 위치를 제거하는 데 사용할 수 있습니다.
        - at : 지정한 정적 리소스를 반환합니다.
    - toH2Console : H2 콘솔 위치를 반환합니다.
</aside>

<br>

중요한 것은 ignoring 뒤에 어떤 방법으로든 설정을 한다면 필터 목록이 비워진다는 것입니다.

즉, 시큐리티 필터가 아무것도 적용이 안되서 성능상 이득이 생깁니다.

# Reference

- 파비콘
    - [https://rlawls1991.tistory.com/entry/index-페이지와-파비콘](https://rlawls1991.tistory.com/entry/index-%ED%8E%98%EC%9D%B4%EC%A7%80%EC%99%80-%ED%8C%8C%EB%B9%84%EC%BD%98)
- 기본 파비콘
    - [https://www.inflearn.com/questions/182043/favicon-질문입니다](https://www.inflearn.com/questions/182043/favicon-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)
    - [https://inflearn.com/questions/300186](https://inflearn.com/questions/300186)
- ignoring 의 다양한 방법
    - [https://ohtaeg.tistory.com/11](https://ohtaeg.tistory.com/11)
    - [https://hyunwook.dev/123](https://hyunwook.dev/123)
    - [https://ojt90902.tistory.com/843](https://ojt90902.tistory.com/843)
- PathRequest
    - [https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/servlet/PathRequest.html](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/servlet/PathRequest.html)