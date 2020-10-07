## java学习-springboot实现自定义WebFilter插件



![https://www.94rg.com](https://ae01.alicdn.com/kf/H7ffc2398a16c4f06b20531828ede40baD.png)

>**郑重声明**: 本文首发于[人工博客](https://www.94rg.com)

### 1、背景
> 需求是这样的，需要自定义一个http插件可以对request的请求按照规则进行改写，对返回结果进行校验和脱敏。提供给其他人使用，要求对接简单。无业务侵入性。

### 2、技术选型
> + springboot
> + filter

### 3、需要解决的问题
> + 要改写request和response就必须自定义实现HttpServletRequestWrapper
>    和HttpServletResponseWrapper
> + 通用方案的话就需要把业务抽象出来以接口的方式让对接方来实现
> + fileter的自动注册与扫描路径的问题，减少对接方的工作量和理解成本

### 4、细节说明
####  4.1 自定义实现HttpServletRequestWrapper

> + 主要是实现一些自己需要关注的属性相关的方法，比如header、param、body、url

```
@Slf4j
public class RgHttpServletRequestWrapper extends HttpServletRequestWrapper {
    private static final String UTF8 = "UTF-8";

    private final String urlKey = "url";
    private final String paramsKey = "params";
    private final String headersKey = "headers";
    private final String bodyKey = "body";
    private final byte[] body;
    private final Map<String, String> customHeaders;
    private final Map<String, String> customParams;
    private final StringBuffer stringBuffer = new StringBuffer();
    private final StringBuilder urlStringBuffer = new StringBuilder();
    private Gson gson = new Gson();

    public RgHttpServletRequestWrapper(
            HttpServletRequest request,
            Map<String, String> cacheMap,
            Map<String, Object> urlMatchCahce) {
        super(request);
        Object urlObj = null == urlMatchCahce ? null : urlMatchCahce.get(urlKey);

        String url = null == urlObj ? null : urlObj.toString();
        Map<String, Object> paramsMap = getMapByKey(urlMatchCahce, paramsKey);
        Map<String, Object> headersMap = getMapByKey(urlMatchCahce, headersKey);
        Map<String, Object> bodyMap = getMapByKey(urlMatchCahce, bodyKey);

        url = DefenferUtil.getRightUrl(url, cacheMap, request, paramsMap);

        urlStringBuffer.append(url);

        this.customHeaders = DefenferUtil.handlerRequestHeader(request, cacheMap, headersMap);
        this.customParams=DefenferUtil.handlerRequestParam(request,cacheMap,paramsMap);

        String sessionStream = getBodyString(request);

        if (null != cacheMap && StringUtils.isNotBlank(sessionStream)) {
            // 替换请求中的参数
            Map<String, Object> map = JSON.parseObject(sessionStream, Map.class);
            map = mergeCacheData(map, cacheMap, bodyMap);
            sessionStream = JSON.toJSONString(map);
        }

        body = sessionStream.getBytes(Charset.forName(UTF8));
    }

    public String getBodyString() {
        return new String(body, Charset.forName(UTF8));
    }

    /**
     * 获取请求Body
     *
     * @param request
     * @return
     */
    private String getBodyString(final ServletRequest request) {
        StringBuilder sb = new StringBuilder();
        InputStream inputStream = null;
        BufferedReader reader = null;
        try {
            inputStream = cloneInputStream(request.getInputStream());
            reader = new BufferedReader(new InputStreamReader(inputStream, Charset.forName(UTF8)));
            String line = "";
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return sb.toString();
    }

    /**
     * Description: 复制输入流</br>
     *
     * @param inputStream
     * @return</br>
     */
    public InputStream cloneInputStream(ServletInputStream inputStream) {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len;
        try {
            while ((len = inputStream.read(buffer)) > -1) {
                byteArrayOutputStream.write(buffer, 0, len);
            }
            byteArrayOutputStream.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
        InputStream byteArrayInputStream =
                new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        return byteArrayInputStream;
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {

        final ByteArrayInputStream bais = new ByteArrayInputStream(body);

        return new ServletInputStream() {

            @Override
            public int read() throws IOException {
                return bais.read();
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
            public void setReadListener(ReadListener readListener) {}
        };
    }

    public void putHeader(String name, String value) {
        this.customHeaders.put(name, value);
    }

    public void removeHeader(String name) {
        this.customHeaders.remove(name);
    }

    @Override
    public String getHeader(String name) {
        // check the custom headers first
        String headerValue = customHeaders.get(name);

        if (headerValue != null) {
            return headerValue;
        }
        // else return from into the original wrapped object
        return ((HttpServletRequest) getRequest()).getHeader(name);
    }

    @Override
    public Enumeration<String> getHeaders(String name) {
        // create a set of the custom header names
        Set<String> set = new HashSet<String>();

        // now add the headers from the wrapped request object
        @SuppressWarnings("unchecked")
        Enumeration<String> e = this.getHeaderNames();
        while (e.hasMoreElements()) {
            // add the names of the request headers into the list
            String n = e.nextElement();
            if (n.equalsIgnoreCase(name)) {
                set.add(getHeader(n));
            }
        }
        // create an enumeration from the set and return
        return Collections.enumeration(set);
    }

    @Override
    public Enumeration<String> getHeaderNames() {
        // create a set of the custom header names
        Set<String> set = new HashSet<String>(customHeaders.keySet());

        // now add the headers from the wrapped request object
        @SuppressWarnings("unchecked")
        Enumeration<String> e = ((HttpServletRequest) getRequest()).getHeaderNames();
        while (e.hasMoreElements()) {
            // add the names of the request headers into the list
            String n = e.nextElement();
            set.add(n);
        }

        // create an enumeration from the set and return
        return Collections.enumeration(set);
    }

  /**
   * The default behavior of this method is to return
   * getParameter(String name) on the wrapped request object.
   *
   * @param name
   */
  @Override
  public String getParameter(String name) {
    if(null!=this.customParams && this.customParams.containsKey(name)){
      return this.customParams.get(name);
    }
    return null;
  }
}

```



#### 4.2、自定义实现过滤器

```
public class DefenderFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}


    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest hsRequest = (HttpServletRequest) request;
        String requestUrl = hsRequest.getRequestURI();
        String contextPath = hsRequest.getContextPath();

        Map<String, Object> urlMatchedMap =
                RgApplicationContextUtil.getBean(UrlMappings.class)
                        .getUrlMatchedMap(requestUrl.replace(contextPath, ""));

        /** 用户的缓存数据 */
        Map<String, String> cacheMap =
                RgApplicationContextUtil.getBean(SystemCacheConfigurationAware.class)
                        .getSystemCache(hsRequest);
        if (null == cacheMap || cacheMap.isEmpty()) {
            log.warn("no cache data,url:{}", requestUrl);
            chain.doFilter(request, response);
        } else {
            RgHttpServletRequestWrapper myHttpServletRequestWrapper =
                    new RgHttpServletRequestWrapper(hsRequest, cacheMap, urlMatchedMap);
            chain.doFilter(myHttpServletRequestWrapper, response);
        }
    }

   
    @Override
    public void destroy() {}
}
```



#### 4.3 解决自注册问题

```
@Configuration
public class DefenderConfiguration {

    @Autowired private DefenderUrlConfigurationAware defenderUrlConfigurationAware;

    @Bean
    public UrlMappings urlMappings() {
        return new UrlMappings(defenderUrlConfigurationAware.getDefenderUrlMaps());
    }

    @Bean
    public FilterRegistrationBean defenderFilter() {

        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new DefenderFilter());
        registrationBean.setOrder(Integer.MIN_VALUE);
        registrationBean.addUrlPatterns("/*");
        return registrationBean;
    }

    @Bean
    @Order()
    public RgApplicationContextUtil rgApplicationContextUtil() {
        RgApplicationContextUtil rgApplicationContextUtil = new RgApplicationContextUtil();
        return rgApplicationContextUtil;
    }
}

```



#### 4.4 增加开启的开关

application上增加这个注解就自动开启了filter的注解功能，是不是很方便

```
@Documented
@Inherited
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import({DefenderConfiguration.class})
public @interface EnableDefender {
}
```



#### 4.5 分享一个spring的工具类

```
@Slf4j
public class RgApplicationContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    public static final ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (Objects.isNull(RgApplicationContextUtil.applicationContext)) {
            RgApplicationContextUtil.applicationContext = applicationContext;
        }
    }

    public static final <T> T getBean(String name, Class<T> requiredType) {
        try {
            return applicationContext.getBean(name, requiredType);
        } catch (Exception e) {
            log.error("get bean error, beanName:{}, class:{}", name, requiredType.getName());
        }
        return null;
    }

    public static final <T> T getBean(Class<T> requiredType) {
        try {
            return applicationContext.getBean(requiredType);
        } catch (Exception e) {
            log.error("get bean error class:{}", requiredType.getName());
        }
        return null;
    }

    public static final <T> T getBean(Class<T> requiredType, Object... args) {
        try {
            return applicationContext.getBean(requiredType, args);
        } catch (Exception e) {
            log.error("get bean error class:{}", requiredType.getName());
        }
        return null;
    }
}
```



------

> 版权声明：本文为人工博客的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
> 本文链接：[https://www.94rg.com/article/1737](https://www.94rg.com/article/1737)