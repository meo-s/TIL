# Architecture

## Security Filters

- `ForceEagerSessionCreationFilter`  
- `ChannelProcessingFilter`  
- `WebAsyncManagerIntegrationFilter`  
- `SecurityContextPersistenceFilter`  
- `HeaderWriterFilter`  
- `CorsFilter`  
- `CsrfFilter`  
- `LogoutFilter`  
- `OAuth2AuthorizationRequestRedirectFilter`  
- `Saml2WebSssoAuthenticationRequestFilter`  
- `X509AuthenticationFilter`  
- `AbstractPreAuthenticatedProcessingFilter`  
- `CasAuthenticationFilter`  
- `OAuth2LoginAuthenticationFilter`  
- `Saml2WebSssoAuthenticationFilter`  
- `UsernamePasswordAuthenticationFilter`  
- `DefaultLoginPageGeneratingFilter`  
- `DefautlLogoutPageGeneratingFilter`  
- `ConcurrentSessionFilter`  
- `DigestAuthenticationFilter`  
- `BearerTokenAuthenticationFilter`  
- `BasicAuthenticationFilter`  
- [`RequestCacheAwareFilter`](#request-cache-aware-filter)  
- `SecurityContextHolderAwareRequestFilter`  
- `JassApiIntegrationFilter`  
- `RememberMeAuthenticationFilter`  
- `AnonymousAuthenticationFilter`  
- `OAuth2AuthorizationCodeGrantFilter`  
- `SessionManagementFilter`  
- [`ExceptionTranslationFilter`](#handling-security-exceptions)  
- `FilterSecurityInterceptor`  
- `SwitchUserFilter`  

## Handling Security Exceptions <a id="handling-security-exceptions"></a>

`ExceptionTranslationFilter`는 `AccessDeniedException`과 `AuthenticationException` 예외 발생에 따른 실행의 흐름을 제어한다.  

![exception-translation-filter](./rsc/exception-translation-filter.png)  

1. `ExceptionTranslationFilter`는 `FilterChain.doFilter(res, resp)`를 호출해 실행 흐름을 위임한다.  
2. `FilterChain.doFilter(res, resp)`로 위임된 실행 흐름 중, 만약 유저가 인증되지 않았거나 `AuthenticationException`이 발생했다면 **인증을 시작**한다.  
    2.1. `SecurityFilterChain`을 초기화한다.  
    2.2. `HttpServletRequest`를 저장한다. 이는 이후에 이어질 인증에 성공하면 동일한 요청을 다시 수행하기 위함이다.  
    2.3. `AuthenticationEntryPoint`를 사용해 클라이언트로부터 자격 증명을 요청한다. 예시로, 로그인 페이지로의 이동 혹은 `WWW-Authenticate` 헤더를 전송해 이루어진다.  
3. 2의 조건에 해당하지 않는다면 `AccessDeniedHandler`가 호출된다.  

> `FilterChain.doFilter(res, resp)`에서 `AuthenticationException` 혹은 `AccessDeniedException`이 발생하지 않는다면 `ExceptionTranslationFilter` 아무런 동작을 하지 않는다.  

## Saving Requests Between Authentication

만약 인증되지 않은 상태에서 인증이 필요한 리소스에 대한 요청이 발생했다면, 인증 이후에 해당 요청을 다시 처리하기 위해 이전의 요청을 저장할 필요가 있다. Spring Security는 `RequestCache`의 구현체를 사용하여 `HttpServletResponse`를 저장한다.  

### RequestCache

요청 재수행을 위한 `HttpServletRequest`는 `ExceptionTranslationFilter`에 의해 `RequestCache`에 저장된다. 이 과정은 `AuthenticationException`이 발생한 이후에 수행된다. 이후에 유저가 인증에 성공하면, `RequestCache`에 저장된 `HttpServletRequest`를 사용하여 이전의 요청을 재수행한다. 이 작업은 `RequestCacheAwareFilter`에서 수행된다.  

### Prevent the Request From Being Saved

요청 저장을 위한 `RequestCache`의 기본값은 `HttpSessionRequestCache`이다. 때때로 요청을 세션에 저장하지 않고 다른 곳에 저장하거나 저장 자체를 원치 않을 수 있다. 요청 저장 방식은 `SecurityFilterChain`을 생성할 때 `requestCache()` 메소드를 사용하여 지정할 수 있다.  

아래의 코드는 요청을 저장하지 않도록 변경하는 예시이다.  

``` java
@Bean
SecurityFilterChain springSecurity(final HttpSecurity http) throws Exception {
    return http
            ...
            .requestCache(cache -> cache.
                    requestCache(new NullRequestCache()))
            .build();
}
```

### `RequestCacheAwareFilter` <a id="request-cache-aware-filter"></a>

`RequestCacheAwareFilter`는 저장된 `HttpServletRequest`가 존재한다면 현재의 요청대신 저장된 `HttpServletRequest`를 다음 애플리케이션 로직으로 전달한다.  

아래는 실제 Spring Security의 `RequestCacheAwareFilter` 구현 코드이다. `doFilter()` 메소드에서 `this.requestCache`에서 저장된 `HttpServletRequest`를 검색하여 만약 존재한다면 이후의 로직에서는 저장된 요청을 사용하도록 저장되어 있던 요청을 전달하는 것을 확인할 수 있다.  

`RequestCacheAwareFilter`가 기본으로 사용하는 `HttpSessionRequestCache`는 단순하게 세션에 요청을 저장한다.  

``` java
public class RequestCacheAwareFilter extends GenericFilterBean {

    private RequestCache requestCache;

    public RequestCacheAwareFilter() {
        this(new HttpSessionRequestCache());
    }

    public RequestCacheAwareFilter(RequestCache requestCache) {
        Assert.notNull(requestCache, "requestCache cannot be null");
        this.requestCache = requestCache;
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest wrappedSavedRequest = this.requestCache.getMatchingRequest((HttpServletRequest) request,
                (HttpServletResponse) response);
        chain.doFilter((wrappedSavedRequest != null) ? wrappedSavedRequest : request, response);
    }

}
```

``` java
public class HttpSessionRequestCache implements RequestCache {
    ...
    @Override
    public void saveRequest(HttpServletRequest request, HttpServletResponse response) {
        ...
        DefaultSavedRequest savedRequest = new DefaultSavedRequest(request, this.portResolver);
        if (this.createSessionAllowed || request.getSession(false) != null) {
            // Store the HTTP request itself. Used by
            // AbstractAuthenticationProcessingFilter
            // for redirection after successful authentication (SEC-29)
            request.getSession().setAttribute(this.sessionAttrName, savedRequest);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug(LogMessage.format("Saved request %s to session", savedRequest.getRedirectUrl()));
            }
        }
        ...
    }
    ...
}
```

## References  

["Architecture"](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters), docs.spring.io  
