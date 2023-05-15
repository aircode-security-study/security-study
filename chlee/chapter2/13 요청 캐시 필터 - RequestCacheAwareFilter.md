# **RequestCacheAwareFilter 란?**

현재 요청과 관련 있는 캐시된 요청이 있는지 찾아서 적용하는 필터입니다.

- 캐시된 요청이 없다면, 현재 요청 처리
- 캐시된 요청이 있다면, 해당 캐시된 요청 처리

# **RequestCacheAwareFilter** 필터를 사용하는 이유?

인증이 안된 상태로 [http://localhost:8080/dashboard](http://localhost:8080/dashboard) 요청을 하게 되면 권한이 없으니 로그인 페이지로 리다이렉트 됩니다. 그리고 로그인을 하게 되면 그제서야 /dashboard 페이지로 이동하게 되죠.

바로 이 때 RequestCacheAwareFilter 가 사용됩니다.

- RequestCacheAwareFilter

<img width="1152" alt="Untitled" src="https://github.com/aircode-security-study/security-study/assets/77683221/abd00f8c-acae-4c3f-b2da-abc2ba396f4e">


로그인 버튼을 클릭하게 되면 /dashboard 로 가야하는데 RequestCacheAwareFilter 에서 캐시된 요청이 꺼내져서 wrappedSavedRequest 에 담기게 됩니다.

- wrappedSavedRequest

<img width="733" alt="Untitled 1" src="https://github.com/aircode-security-study/security-study/assets/77683221/651142e8-54d0-40c2-9bb5-520724cae69c">


원래 dashboard 로 가려고 했는데 예외가 발생해서 로그인 처리를 하게 되었잖아요?

dashboard로 가려던 요청을 잠시 wrappedSavedRequest 에다가 캐시해놓은 것입니다.

원래 해야하는 요청을 할 수 있게끔 도와주는 필터인 것이죠.

사실 크게 설정을 하거나 신경을 쓸 필요가 없는 필터입니다.