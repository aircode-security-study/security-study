![waving](https://capsule-render.vercel.app/api?type=waving&height=150&text=security-study&fontAlign=72&fontAlignY=40&fontSize=60&color=gradient&fontColor=FFFFFF)

## About
- [ Spring Security ]를 공부, 발표, 토론하고 정리해둔 공간입니다.  
- 주목적은 **학습**이지만, 향후 다른 개발자들과 **소통**하여 좋은 영향을 주고 받을 수 있게 되는 것이 또 하나의 목표입니다.

<!--
## Guide

> [스터디 규칙 :bulb:]()   

> [github 작업 가이드 :guide_dog:]()
-->

## Contents
4/11 ~ 4/18 (완료)
- 섹션 0. 스프링 시큐리티: 폼 인증
  - 폼 인증 예제 살펴보기
  - 스프링 웹 프로젝트 만들기
  - 스프링 시큐리티 연동
  - 스프링 시큐리티 설정하기
  - 스프링 시큐리티 커스터마이징: 인메모리 유저 추가
  - 스프링 시큐리티 커스터마이징: JPA 연동
  - 스프링 시큐리티 커스터마이징: PasswordEncoder
  - 스프링 시큐리티 테스트 1부
  - 스프링 시큐리티 테스트 2부

4/18 ~ 4/25 (완료)
- 섹션 1. 스프링 시큐리티: 아키텍처
  - SecurityContextHolder와 Authentication
  - AuthenticationManager와 Authentication
  - ThreadLocal
  - Authentication과 SecurityContextHodler
  - 스프링 시큐리티 필터와 FilterChainProxy
  - DelegatingFilterProxy와 FilterChainProxy
  - AccessDecisionManager 1부
  - AccessDecisionManager 2부
  - FilterSecurityInterceptor
  - ExceptionTranslationFilter
  - 스프링 시큐리티 아키텍처 정리

4/25 ~ 5/10 (완료)
- 섹션 2. 웹 애플리케이션 시큐리티
  - 스프링 시큐리티 ignoring() 1부
  - 스프링 시큐리티 ignoring() 2부
  - Async 웹 MVC를 지원하는 필터: WebAsyncManagerIntegrationFilter
  - 스프링 시큐리티와 @Async
  - SecurityContext 영속화 필터: SecurityContextPersistenceFilter
  - 시큐리티 관련 헤더 추가하는 필터: HeaderWriterFilter
  - CSRF 어택 방지 필터: CsrfFilter
  - CSRF 토큰 사용 예제
  - 로그아웃 처리 필터: LogoutFilter
  - 폼 인증 처리 필터: UsernamePasswordAuthenticationFilter
  - 로그인/로그아웃 폼 페이지 생성해주는 필터: DefaultLogin/LogoutPageGeneratingFilter
  - 로그인/로그아웃 폼 커스터마이징

5/10 ~ 5/16 (진행중)
- 섹션 2. 웹 애플리케이션 시큐리티
  - Basic 인증 처리 필터: BasicAuthenticationFilter
  - 요청 캐시 필터: RequestCacheAwareFilter
  - 시큐리티 관련 서블릿 스팩 구현 필터: SecurityContextHolderAwareRequestFilter
  - 익명 인증 필터: AnonymousAuthenticationFilter
  - 세션 관리 필터: SessionManagementFilter
  - 인증/인가 예외 처리 필터: ExceptionTranslationFilter
  - 인가 처리 필터: FilterSecurityInterceptor
  - 토큰 기반 인증 필터 : RememberMeAuthenticationFilter
  - 커스텀 필터 추가하기

## Contributors
<p>
<a href="https://github.com/chanhl22">
  <img src="https://avatars.githubusercontent.com/u/77683221?v=4" width="100">
</a>
<a href="https://github.com/jeongwoogeun">
  <img src="https://avatars.githubusercontent.com/u/50163299?v=4" width="100">
</a>
<a href="https://github.com/yoonwooseong">
  <img src="https://avatars.githubusercontent.com/u/57824259?v=4" width="100">
</a>
</p>

## Reference
- [인프런 - 스프링 시큐리](https://www.inflearn.com/course/%EB%B0%B1%EA%B8%B0%EC%84%A0-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0#)
