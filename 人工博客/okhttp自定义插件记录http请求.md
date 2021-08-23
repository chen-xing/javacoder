### okhttp自定义插件记录http请求

### 1、定义插件

```
package tech.chenxing.configuration;

import okhttp3.*;
import okio.Buffer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class OkHttpLoggingInterceptor implements Interceptor {
    private static final Logger logger = LoggerFactory.getLogger("request_logger");

    @Autowired private SystemConfig systemConfig;

    @Override
    public Response intercept(Chain chain) throws IOException {
        long startTime = System.currentTimeMillis();
        Request request = chain.request();
        String requestParam = null;
        String responseData = null;
        Exception exception = null;

        try {
            Response response = chain.proceed(request);
            if (!systemConfig.isLOGGER_REQUEST_ENABLED()) {
                return response;
            }
            requestParam = parseRequestParam(request.body());
            ResponseBody responseBody = response.body();
            MediaType mediaType = responseBody.contentType();
            responseData = parseResponseData(responseBody);
            return response.newBuilder().body(ResponseBody.create(mediaType, responseData)).build();
        } catch (Exception e) {
            exception = e;
            throw e;
        } finally {
            saveRequestLog(
                    request.url().url().toString(),
                    requestParam,
                    responseData,
                    System.currentTimeMillis() - startTime,
                    exception);
        }
    }

    /**
     * 解析请求参数
     *
     * @param requestBody
     * @return
     */
    private String parseRequestParam(RequestBody requestBody) {
        Buffer buffer = new Buffer();
        try {
            requestBody.writeTo(buffer);
            return buffer.readUtf8();
        } catch (IOException e) {
            logger.error("parse feign request param exception.", e);
            return "";
        }
    }

    /**
     * 解析响应结果
     *
     * @param responseBody
     * @return
     */
    private String parseResponseData(ResponseBody responseBody) {
        try {
            return responseBody.string();
        } catch (IOException e) {
            logger.error("parse feign response data exception.", e);
            return "";
        }
    }

    /**
     * 保存请求日志
     *
     * @param requestUrl
     * @param inputStr
     * @param outputStr
     * @param time
     * @param e
     */
    private void saveRequestLog(
            String requestUrl, String inputStr, String outputStr, long time, Exception e) {
        if (!systemConfig.isLOGGER_REQUEST_ENABLED()) {
            return;
        }
        logger.info(
                "RpcRequestUrl : {}, Time : {}ms, Input : {}, Output : {}. {}",
                requestUrl,
                time,
                inputStr,
                outputStr,
                null != e ? "Exception : " : "",
                e);
    }
}

```



### 2、注入插件对象

```
package tech.chenxing.configuration;

import okhttp3.ConnectionPool;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
public class OkHttpLoggingConfiguration {
    @Bean
    @ConditionalOnBean(OkHttpLoggingInterceptor.class)
    public okhttp3.OkHttpClient rgOkHttpClient(
            @Autowired OkHttpLoggingInterceptor okHttpLoggingInterceptor) {
        okhttp3.OkHttpClient.Builder ClientBuilder =
                new okhttp3.OkHttpClient.Builder()
                        .connectionPool(new ConnectionPool(256, 600, TimeUnit.SECONDS))
                        .addInterceptor(okHttpLoggingInterceptor);
        return ClientBuilder.build();
    }
}

```

