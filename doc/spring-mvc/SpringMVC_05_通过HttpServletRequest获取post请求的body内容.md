[toc]







# 前言

## 推荐阅读

> - [Get the POST request body from HttpServletRequest](https://stackoverflow.com/questions/8100634/get-the-post-request-body-from-httpservletrequest)
> - [解决springboot filter 获取提交过来的参数](https://www.phpsong.com/3840.html)
> - [SpringBoot项目, 拦截器获取Post方法的请求body](https://blog.csdn.net/mazaiting/article/details/93930333)
> - [HttpServletRequest.getInputStream()多次读取问题](https://www.jianshu.com/p/85feeb30c1ed)
> - 





# 一、通过HttpServletRequest获取post请求的body内容

Get请求参数是放在HttpServletRequest 的parameterMap中的，可以通过如下方式获取get请求参数

```java
request.getParameterMap()
request.getParameter("parameterName")
```



Post请求参数，是以字节流的形式放在请求体中的，HttpServletRequest 没有提供直接通过参数名获取Post请求参数的方法。

要通过HttpServletRequest 获取Post请求参数，只能选择从HttpServletRequest 中读取字节流，然后转成字符串了。

```java
 private String getBody(HttpServletRequest request) throws IOException {
    InputStream in = request.getInputStream();
    BufferedReader br = new BufferedReader(new InputStreamReader(in, Charset.forName("UTF-8")));
    StringBuffer sb = new StringBuffer("");
    String temp;
    while ((temp = br.readLine()) != null) {
        sb.append(temp);
    }
    if (in != null) {
        in.close();
    }
    if (br != null) {
        br.close();
    }
    return sb.toString();
}
```



直接这样读取，会导致接下来的程序中无法继续通过HttpServletRequest获取Post请求体







### 1.HttpServletRequestWrapper

```java
public class HttpRequestWrapper extends HttpServletRequestWrapper {
    private final String body;

    public HttpRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        body = IOUtils.toString(request.getReader());
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(getBody().getBytes());
        ServletInputStream servletInputStream = new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return byteArrayInputStream.read();
            }

            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener listener) {
            }

        };
        return servletInputStream;
    }

    public String getBody() {
        return this.body;
    }
}
```





### 2.AclAfterAuthenticationFilter

```
@Component
@Slf4j
public class AclAfterAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    AclRule aclRule;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException{
        HttpRequestWrapper requestWrapper = new HttpRequestWrapper(request);
        if (!aclRule.accept(requestWrapper)) {
            throw new BaseException( "Access is denied!", HttpStatus.FORBIDDEN);
        }

        filterChain.doFilter(requestWrapper, response);
    }



}

```



### 3.PermAclRule

```java
    private Integer obtainAppId(HttpRequestWrapper requestWrapper) {
        if (HttpMethod.GET.matches(requestWrapper.getMethod())) {
            String appId = requestWrapper.getParameter(KEY_APP_ID);
            if (!StringUtils.isEmpty(appId)) {
                return Integer.valueOf(appId);
            }
            return null;
        } else {
            String requestBody = requestWrapper.getBody();
            if (JSON.isValid(requestBody)) {
                JSONObject jsonObject = JSON.parseObject(requestBody);
                Integer appId = jsonObject.getInteger(KEY_APP_ID);
                if (!ObjectUtils.isEmpty(appId)) {
                    return appId;
                }
            }
        }

        return null;
    }
```



























