# Redis的JWT升级

在技术派的1.0设计中，token内部包含了用户的id。然后在拦截器中解析。当用户登录时生成一个token到用户的Cookie中，之后用户访问网页就无需登录。直到用户执行登录操作，清理token或者token过期，这样用户才算是退出登录。这样看来，token的过期权限几乎在用户的手中，因为服务端只负责生成token，而不会管理。

在技术派的2.0设计中，使用了SessionHelper来存储用户的token，当得到用户请求中的token时，尝试从Redis来获取，假如token不存在或者已经过期，则需要重新登录。这种设计让token的过期权限移动到了服务端的手里。

### 我的想法

JWT的最大优点应该是无状态和安全，所以完全没有必要将token存储到Redis中。我推荐使用双token机制。

规定：用户存在两个token，都会存储用户信息。

-   accessToken：有效期1小时
-   refreshToken：有效期7天

前端：

-   使用accessToken请求资源：假如成功则返回；假如失败，进入下一步。
-   使用refreshToken请求资源：假如失败则跳转登录界面；假如成功，后端会返回新的accessToken和refreshToken，更新本地的token。

后端：

-   检查token是否有效：假如无效则不通过；假如有效进入下一步。
-   检查token的类型，假如为refreshToken则重新生成两个token在Authorization中。

该阶段可以在Interceptor执行，也可以在Filter执行，但推荐使用Spring Security框架实现。我在【版本1.0】的【2 权限管理】中提到了Spring Security，你可以在那篇文章看到关于Spring Security的大致设计。



