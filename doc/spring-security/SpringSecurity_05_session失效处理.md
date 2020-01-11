[TOC]





# 前言

```java

/**
 * @See https://stackoverflow.com/questions/11242174/handle-session-expired-event-in-spring-based-web-application
 */
public class AjaxAwareAuthenticationEntryPoint extends LoginUrlAuthenticationEntryPoint {
    public AjaxAwareAuthenticationEntryPoint(final String loginFormUrl) {
        super(loginFormUrl);
    }

    @Override
    public void commence(final HttpServletRequest request, final HttpServletResponse response, final AuthenticationException authException)
            throws IOException, ServletException {
        if ("xmlhttprequest".equalsIgnoreCase(request.getHeader("x-requested-with"))) {
            response.sendError(403, "Forbidden");
        } else {
            super.commence(request, response, authException);
        }
    }
}

```







https://stackoverflow.com/questions/11242174/handle-session-expired-event-in-spring-based-web-application



https://www.programcreek.com/java-api-examples/index.php?api=org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint





# 参考资料

1. []
2. [《Java实战开发》利用spring-security解决CSRF问题，通过重写CsrfFilter 过滤掉指定方法](https://blog.csdn.net/qq_39338799/article/details/85274706)
3. 

