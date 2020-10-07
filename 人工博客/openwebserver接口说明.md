

## openwebserver接口说明

### 1、服务定位

> + 解决前端与网关对接需要携带appId和appSecret的问题
> + 保护敏感参数，让前端只需知道参数的key，而不需要知道实参就可以业务的调用
> + 辅助业务，比如加密字符的解密，签署身份的判断，登录拦截，封装了一套缓存的管理接口给前端



### 2、接口的说明

| 接口路径               | 参数                       | 返回值              | 特别说明                                     | 请求示例                                             |
| :--------------------- | :------------------------- | :------------------ | -------------------------------------------- | ---------------------------------------------------- |
| /login                 | 短链中包含的context参数    | 解密context中的参数 |                                              |                                                      |
| /saasLogin             | 短链中包含的context参数    | 解密context中的参数 | 这个是需要实名签署的时候调用的               | 查看页面增加一个参数 showSignButton=true             |
| /saasLogout            | 无                         |                     | 退出当前的登录用户                           |                                                      |
| /refreshCookies        | sesionId                   |                     | 用于解决跨浏览器登录的session复制的问题      |                                                      |
| /getLoginInfo          |                            |                     | 获取当前的登录用户的登录信息                 |                                                      |
| /setCacheData          | url中包含key,value在body中 |                     | 默认为10分钟                                 |                                                      |
| /getCacheData          | url中包含key               |                     |                                              | /openwebserver/getCacheData?cacheKey=xxx             |
| /updateCacheData       | url中包含key,value在body中 |                     | 与setCacheData类似，不过默认的缓存时间是10天 | /openwebserver/updateCacheData?cacheKey=xxx          |
| /clearCacheData        | url中包含key               |                     | 清理无效的缓存                               | /openwebserver/clearCacheData?cacheKey=xxx           |
| /getEncryptionStrValue | url中包含key               |                     | 获取context中的加密值                        | /openwebserver/getEncryptionStrValue?variableKey=xxx |