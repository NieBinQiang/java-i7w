## JWT
JWT(Json web token)是一种用于双方之间**传递安全信息的**、**简洁的**、**URL安全的**表述性声明规范。
- **简洁(Compact)**: 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
- **自包含(Self-contained)**：负载中包含了所有用户所需要的信息，避免了多次查询数据库
## JWT主要应用场景
- 一旦用户完成了登陆，在接下来的**每个请求中都包含JWT**，可以用来验证用户身份以及**对路由、服务和资源的访问权限进行验证**。
- 由于它的开销非常小，可以轻松的在不同域名的系统中传递，所以目前在**单点登录**（SSO）中比较广泛的使用了该技术。

## JWT的结构
JWT包含了使用“.”分隔的三部分： **Header** 头部 **Payload** 负载 **Signature** 签名。结构如下：**Header.Payload.Signature**。
### Header
- 在header中通常包含了两部分：token类型和采用的加密算法。如：{ "alg": "HS256", "typ": "JWT"}
- 对这部分内容使用 **Base64Url** 编码组成了JWT结构的第一部分。
### Payload
Token的第二部分是**负载**，它包含了**Claim**， Claim是一些实体（通常指的用户）的**状态和额外的元数据**，有三种类型的claim：reserved, public 和 private。负载也需要经过**Base64Url**编码后作为JWT结构的第二部分。
- **Reserved claims**: 这些claim是JWT**预先定义**的，在JWT中并不会强制使用它们，而是**推荐使用**，常用的有 iss（签发者）,exp（过期时间戳）, sub（面向的用户）, aud（接收方）, iat（签发时间）。
- **Public claims**：根据需要定义自己的字段，注意应该避免冲突
- **Private claims**：这些是自定义的字段，可以用来在双方之间交换信息 负载使用的例子：{ "sub": "1234567890", "name": "John Doe", "admin": true} 
## Signature
创建签名需要使用**编码后**的header和payload以及一个**秘钥**，使用header中指定签名算法进行签名。

如使用HMAC SHA256算法，则用 HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret) 的形式创建签名。

## JWT应用
- 在身份鉴定的实现中，传统方法是在服务端存储一个session，给客户端返回一个cookie
- 使用JWT之后，当用户使用它的认证信息登陆系统之后，会返回给用户一个JWT，**用户只需要本地保存该token**（**通常使用local storage**，也可以使用cookie）即可。
- 当用户希望访问一个受保护的路由或者资源的时候，通常应该**在Authorization头部使用Bearer模式添加JWT**，其内容看起来是：Authorization: Bearer <token>
- 服务端的保护路由将会**检查请求头Authorization中的JWT信**息，如果合法，则允许用户的行为。

![image](https://segmentfault.com/image?src=http://source.aicode.cc/markdown/jwt-diagram.png&objectId=1190000005047525&token=fc83f4c0cf107cabeafd6a449cd49762)

## 项目应用流程
### 前端
1. 前端vue每次路由调转前，判断localstorge中是否包含token；否则重定向到login页面
2.用户登陆成功后，将返回的token保存在localStorge里
3. 每次调用axios的http请求时，设置拦截器，将config.headers['authentication']设置保存的token值  
### 后端
1. 使用JwtBuilder，设置id、IssueDate，设置claim（accessToken实体，包含用户名、tokenType、开始日期、截止时间），然后调用builder.signWith生成JWT字符串。
2. 生成的同时，以jwt的token为Key，userId作为value的键值对储存在后台缓存中。



