### **基于token的用户认证**

#### 	**一、必须要了解的Cookie和Session.**

##### **1.1 第一层楼**-什么是Cookie和Session？

​	**什么是 Cookie**

​	HTTP Cookie（也叫 Web Cookie或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为了可能。

Cookie 主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

  **什么是 Session**

​	Session 代表着服务器和客户端一次会话的过程。Session 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。当客户端关闭会话，或者 Session 超时失效时会话结束。

​	Cookie一般会存sessionId,然后发送到服务端，服务端根据SessionId可以查到存储的用户的Session。



##### **1.2 第二层楼**-Cookie和Session有什么不同？

- 作用范围不同，Cookie 保存在客户端（浏览器），Session 保存在服务器端。
- 存取方式的不同，Cookie 只能保存 ASCII，Session 可以存任意数据类型，一般情况下我们可以在 Session 中保持一些常用变量信息，比如说 UserId 等。
- 有效期不同，Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭或者 Session 超时都会失效。
- 隐私策略不同，Cookie 存储在客户端，比较容易遭到不法获取，早期有人将用户的登录名和密码存储在 Cookie 中导致信息被窃取；Session 存储在服务端，安全性相对 Cookie 要好一些。
- 存储大小不同， 单个 Cookie 保存的数据不能超过 4K，Session 可存储数据远高于 Cookie。



##### **1.3 第三层楼**-为什么需要Cookie和Session？他们有什么关联？

​	为什么需要 Cookie 和 Session，他们有什么关联？

​	说起来为什么需要 Cookie ，这就需要从浏览器开始说起，我们都知道浏览器是没有状态的(HTTP 协议无状态)，这意味着浏览器并不知道是张三还是李四在和服务端打交道。这个时候就需要有一个机制来告诉服务端，本次操作用户是否登录，是哪个用户在执行的操作，那这套机制的实现就需要 Cookie 和 Session 的配合。

那么 Cookie 和 Session 是如何配合的呢？我画了一张图大家可以先了解下。

​	用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建创建对应的 Session ，请求返回时将此 Session 的唯一标识信息 SessionID 返回给浏览器，浏览器接收到服务器返回的 SessionID 信息后，会将此信息存入到 Cookie 中，同时 Cookie 记录此 SessionID 属于哪个域名。

​	当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 Cookie 信息，如果存在自动将 Cookie 信息也发送给服务端，服务端会从 Cookie 中获取 SessionID，再根据 SessionID 查找对应的 Session 信息，如果没有找到说明用户没有登录或者登录失效，如果找到 Session 证明用户已经登录可执行后面操作。

​	根据以上流程可知，SessionID 是连接 Cookie 和 Session 的一道桥梁，大部分系统也是根据此原理来验证用户登录状态。

​	三层楼的内容，大部分同学可以讲清楚。



##### **1.4 第四层楼**-如果禁用Cookie，解决方案有哪些？

​	既然服务端是根据 Cookie 中的信息判断用户是否登录，那么如果浏览器中禁止了 Cookie，如何保障整个机制的正常运转。

​	第一种方案，每次请求中都携带一个 SessionID 的参数，也可以 Post 的方式提交，也可以在请求的地址后面拼接 `xxx?SessionID=123456...`。

​	第二种方案，Token 机制。Token 机制多用于 App 客户端和服务器交互的模式，也可以用于 Web 端做用户状态管理。

​	Token 的意思是“令牌”，是服务端生成的一串字符串，作为客户端进行请求的一个标识。Token 机制和 Cookie 和 Session 的使用机制比较类似。

​	当用户第一次登录后，服务器根据提交的用户信息生成一个 Token，响应时将 Token 返回给客户端，以后客户端只需带上这个 Token 前来请求数据即可，无需再次登录验证。

​	四层楼的内容，一部分同学可以讲清楚。



##### **1.5 第五层楼**-如何考虑分布式Session问题？

​	在互联网公司为了可以支撑更大的流量，后端往往需要多台服务器共同来支撑前端用户请求，那如果用户在 A 服务器登录了，第二次请求跑到服务 B 就会出现登录失效问题。

分布式 Session 一般会有以下几种解决方案：

- Nginx ip_hash 策略，服务端使用 Nginx 代理，每个请求按访问 IP 的 hash 分配，这样来自同一 IP 固定访问一个后台服务器，避免了在服务器 A 创建 Session，第二次分发到服务器 B 的现象。
- Session 复制，任何一个服务器上的 Session 发生改变（增删改），该节点会把这个 Session 的所有内容序列化，然后广播给所有其它节点。
- 共享 Session，服务端无状态话，将用户的 Session 等信息使用缓存中间件来统一管理，保障分发到每一个服务器的响应结果都一致。

建议采用第三种方案。



##### **1.6 第六层楼**-如何解决跨域问题？

​	如何解决跨域请求？Jsonp 跨域的原理是什么？

​	说起跨域请求，必须要了解浏览器的同源策略，同源策略/SOP（Same origin policy）是一种约定，由 Netscape 公司 1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到 XSS、CSFR 等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个 ip 地址，也非同源。

解决跨域请求的常用方法是：

- 通过代理来避免，比如使用 Nginx 在后端转发请求，避免了前端出现跨域的问题。
- 通过 Jsonp 跨域
- 其它跨域解决方案

​          重点谈一下 Jsonp 跨域原理。浏览器的同源策略把跨域请求都禁止了，但是页面中的 `<script><img><iframe>`标签是例外，不受同源策略限制。Jsonp 就是利用 `<script>` 标签跨域特性进行跨域数据访问。

JSONP 的理念就是，与服务端约定好一个回调函数名，服务端接收到请求后，将返回一段 Javascript，在这段 Javascript 代码中调用了约定好的回调函数，并且将数据作为参数进行传递。当网页接收到这段 Javascript 代码后，就会执行这个回调函数，这时数据已经成功传输到客户端了。

JSONP 的缺点是：它只支持 GET 请求，而不支持 POST 请求等其他类型的 HTTP 请求。



#### **二、SSO-单点登录**

##### 2.1 单点登录流程

​	单点登录英文全称Single Sign On，简称就是SSO。它的解释是：**在多个应用系统中，只需要登录一次，就可以访问其他相互信任的应用系统。**如下图：Application1登录了SSO服务，下次再访问Application2服务的时候就不需要再次登录了，可直接访问。

![image](https://yqfile.alicdn.com/721f02ebe06639e6232b59535d6423db75086693.png)



上图是CAS官网上的标准流程，具体流程如下：

1. 用户访问app系统，app系统是需要登录的，但用户现在没有登录。
2. 跳转到CAS server，即SSO登录系统，**以后图中的CAS Server我们统一叫做SSO系统。** SSO系统也没有登录，弹出用户登录页。
3. 用户填写用户名、密码，SSO系统进行认证后，将登录状态写入SSO的session，浏览器（Browser）中写入SSO域下的Cookie。
4. SSO系统登录完成后会生成一个ST（Service Ticket），然后跳转到app系统，同时将ST作为参数传递给app系统。
5. app系统拿到ST后，从后台向SSO发送请求，验证ST是否有效。
6. 验证通过后，app系统将登录状态写入session并设置app域下的Cookie。

至此，跨域单点登录就完成了。以后我们再访问app系统时，app就是登录的。接下来，我们再看看访问app2系统时的流程。

1. 用户访问app2系统，app2系统没有登录，跳转到SSO。
2. 由于SSO已经登录了，不需要重新登录认证。
3. SSO生成ST，浏览器跳转到app2系统，并将ST作为参数传递给app2。
4. app2拿到ST，后台访问SSO，验证ST是否有效。
5. 验证成功后，app2将登录状态写入session，并在app2域下写入Cookie。

这样，app2系统不需要走登录流程，就已经是登录了。SSO，app和app2在不同的域，它们之间的session不共享也是没问题的。

**有的同学问我，SSO系统登录后，跳回原业务系统时，带了个参数ST，业务系统还要拿ST再次访问SSO进行验证，觉得这个步骤有点多余。他想SSO登录认证通过后，通过回调地址将用户信息返回给原业务系统，原业务系统直接设置登录状态，这样流程简单，也完成了登录，不是很好吗？**

**其实这样问题时很严重的，如果我在SSO没有登录，而是直接在浏览器中敲入回调的地址，并带上伪造的用户信息，是不是业务系统也认为登录了呢？这是很可怕的。**

##### 2.2 单点登录总结

单点登录（SSO）的所有流程都介绍完了，原理大家都清楚了。总结一下单点登录要做的事情：

- **单点登录（SSO系统）是保障各业务系统的用户资源的安全 。**
- **各个业务系统获得的信息是，这个用户能不能访问我的资源。**
- **单点登录，资源都在各个业务系统这边，不在SSO那一方。 用户在给SSO服务器提供了用户名密码后，作为业务系统并不知道这件事。 SSO随便给业务系统一个ST，那么业务系统是不能确定这个ST是用户伪造的，还是真的有效，所以要拿着这个ST去SSO服务器再问一下，这个用户给我的ST是否有效，是有效的我才能让这个用户访问。**



#### **三、基于token的用户验证**

背景
		传统的用户验证是基于session自身的特性实现，当用户提交登陆请求，后台验证通过后，会在session中留下用户的信息，用于识别当前用户在客户端登陆了。通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。因为认证的记录是保存在内存中，意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

##### **3.1、基于Token的用户验证**

​		基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

​		常见验证流程:

1.用户提交用户名、密码到服务器后台
2.后台验证用户信息的正确性
3.若用户验证通过，服务器端生成Token，返回到客户端
4.客户端保存Token，再下一次请求资源时，附带上Token信息
5.服务器端（一般在拦截器中进行拦截）验证Token是否由服务器签发的
6.若Token验证通过，则返回需要的资源

##### **3.2、JSON Web Token（JWT）**

​		JWT是一种通用的规范，它定义了Token的生成方式。

###### **3.2.1 JWT的格式**

一个完整的Token的形式：

```java
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1bmlzaW1zIiwicGVyaW9kIjo2MDAwMCwiZXhwIjoxNTMxODI5NzI5LCJ1c2VySWQiOiIwZjg3LTQxYzI2ZGExMzUxNjA2MDhkNTYtIiwiaWF0IjoxNTMxODI5NjA5fQ.RZCUmxfaDgPxhocCkomSDcUOwLNYUW3Hgu-ufi0mJZNlurGSQHex0CokiUqRTfhQo0G8VJuYDzjeUklHN2pAdA
```


由“.”符号拼接，共有三部分信息组成。

JWT由三部分组成：Header、Payload、Signature。

```json
JWT:{
    header:{ // 头部
        alg:'HS256' // 算法声明
    },
    payload:{ // 数据
        exp:'1532180906', // 过期时间
        userId:'xxxx',
        iss:'xxx'
    },
    signature:'' // 签名
}
```

###### **3.2.2 Header（头部）**

**jwt的头部承载两部分信息：**

- 声明类型，这里是jwt
- 声明加密的算法 通常直接使用 HMAC SHA256

###### **3.2.3 Payload（数据）**

这部分包含了一些通用的信息，以及用户自定义的信息。通常在做用户验证时，会放置userId等信息。
通用信息组成（不强制全部使用）：

- iss: jwt签发者
- sub: jwt所面向的用户
- aud: 接收jwt的一方
- exp: jwt的过期时间，这个过期时间必须要大于签发时间
- nbf: 定义在什么时间之前，该jwt都是不可用的.
- iat: jwt的签发时间
- jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
  

###### **3.2.4 Signature（签证）**

这部分信息是一个签证信息，由算法进行加密，它由三部分信息组成：

- Header（base64编码后的字符串）
- Payload（base64编码后的字符串）
- Secret
  签证信息的生成方式：

```
将编码后的Header、Payload由“.”符号进行拼接，将拼接后的字符串使用加密算法进行加密
```

加密算法的选择：尽量选择哈希散列解密方式，避免被容易的暴力破解。
**备注：**密钥需要妥善保存，这是验证Token有效性的唯一标识。

###### **3.2.5 Token的生成方式**

```java
{{base64Encode(Header)}}.{{base64Encode(Payload)}}.{{Signature}}
```

将编码后的Header、编码后的Payload以及Signature由符号“.”进行拼接，形成最终的Token。

##### **3.3、JWT的实现**

​		JWT提供了Token的生成规范，没有现成的JAR包可以引用。可以自己手动实现，也可以使用开源项目。推荐一个开源的JWT实现，使用起来挺方便的。Java版本的JWT实现

###### **3.3.1 生成一个Token**

```java
   private Key getKey(){
        if(ObjectUtils.isEmpty(key)){
            key = MacProvider.generateKey();
        }
        return key;
    }
	@Override
    public String generatorToken(String userId){
        Key key = getKey();
    Long currentTime = System.currentTimeMillis();
    Long activeTime = 30 * 60000L; //

    Map<String, Object> claims = new HashMap<>();
    claims.put(USERID, userId);
    claims.put(PERIOD, activeTime / 2); // 有效时间

    String compactJws = Jwts.builder()
            .setClaims(claims)  // 自定义数据
            .setSubject("unisims")  //  
            .setIssuedAt(new Date(currentTime)) // 签发时间
            .setExpiration(new Date(currentTime + activeTime)) // 超时时间
            .signWith(SignatureAlgorithm.HS512, key) // 签名
            .compact();

    return compactJws;
}   
```
###### **3.3.2 验证Token**

```java
@Override
public JWTCheckResult validateToken(String token) {
        JWTCheckResult result = new JWTCheckResult();   
	if(StringUtils.isEmpty(token)){
        result.setStatus(JWTEnum.EMPTY.getStatus());
        result.setMessage(JWTEnum.EMPTY.getMessage());
        return result;
    }

    try {
        Key key = getKey();
        Jws<Claims> jwt = Jwts.parser().setSigningKey(key).parseClaimsJws(token);

    } catch (SignatureException e) {
        result.setStatus(JWTEnum.ERROR.getStatus());
        result.setMessage(JWTEnum.ERROR.getMessage());

        System.out.println(e);
    }catch(ExpiredJwtException e){
        result.setStatus(JWTEnum.TIMEOUT.getStatus());
        result.setMessage(JWTEnum.TIMEOUT.getMessage());

        System.out.println(e);
    }catch(Exception e){
        result.setStatus(JWTEnum.ERROR.getStatus());
        result.setMessage(JWTEnum.ERROR.getMessage());
        System.out.println(e);
    }
    return result;
}
```
#### **四、Token在用户验证中真正的使用**

​	第三节中讲了Token的生成与简单验证，但是，在项目中真正真正使用时，仅有这些是不够的。

##### **4.1 Token的第一次签发**

​		当服务端收到登陆请求，需要在后台验证账户信息。若是验证通过，则生成Token，除了固定的信息外，还会放置一些与用户相关的信息，通常就是放置用户的ID。
将生成的Token返回前端，一般是将Token放置在response的header中。同时，浏览器在下一次请求也是将Token放置在请求头中。

##### **4.2 Token的验证**

​		客户端请求资源时，需要对客户端的用户进行验证。有些请求是不需要的验证用户（比如登陆请求，不需要验证用户，此时本就没有用户登陆），有些资源是需要验证的，所以需要对URL进行区分。

​		Teken的验证我选择在拦截器中实现（HandlerInterceptor），在这里对URL进行拦截。首先判断是否是需要验证的URL，然后对Token进行验证。对于Token的验证分为成功验证、无效验证、超时验证、刷新处理、主动失效处理。

**Token验证方式的选择：**

第一种验证方式：利用session
		在服务器端，生成Token的同时将Token存入session。当收到客户端上传的Token时，将它与session中的进行比对，一致则合法，验证通过。否则，返回验证失败，前端跳转到登陆页面。

第二种验证方式：算法验证
		当收到客户端上传的Token时，对Token的前两部分进行加密，比对加密结果是否与第三部分相同，相同则验证通过。否则，返回验证失败，前端跳转到登陆页面。

```
备注：为了项目的扩展性考虑，采用第二种方式进行验证。第一种方式依赖于Session，做负载均衡时，当请求被转发到另一台服务器时，由于Session中没有Token信息，会造成成验证不通过。
```

成功验证：
保证Token是正确的有两个因素：第一，这个Token能够正确被Token识别，即这个Token的确是由自身的后台签发的；第二，这个Toen没有过期。

```
一般的，只要这个能够被后台使用自身的密钥+算法正确解密，即可认为这个Token是由自身签发的。
```

Token的失效：

​		一般都会为Token添加有效时间区间，即在某个时间区间这个Token是有效的，过了这个时间点后，即认为这是一个不可信任的Token。

上文讲述到的那个开源项目中，已经做好了无效、超时Token验证的接口，不需要我们手动去实现。

无效验证：
		不能被正确解析的Token、以及超时的Token，既是验证不通过。

超时验证：
		默认每一个Token都会为它添加有效时间，当超过生效时间，则认为此Token验不通过。
一般的，在生成Token时，会在Payload中设置过期时间，在拦截器中，根据这个时间验证是否超时。

```
上面提到 开源项目中已经自动处理的超时的处理，不需要认为的再次处理，仅仅只需要设置一个过期时间即可。
```

刷新处理：
		因为业务的需要，不能要求每次Token超时后，用户再次登陆。所以，需要在用户无感知情况下自动刷新Token，避免用户再次发起登陆请求。

第一种方式：通过算法计算
		将过期周期认为是2T，当在T-2T时间区间时，认为此时需要刷新Token。当此Token验证通过，则在后台生成新的Token，随着header返回到前端，前端自己去刷新Token。

​		这种方式有个缺点，即只要不是超时的Token_A，依然可以正常的请求后台资源。即，假如在T-2T时间区间刷新Token，产生了新的Token_B，此时Token_A依然是有效的Token。

第二种方式：配合内存、数据库刷新Token
预先将生成的Token存储在内存、或数据库中，在拦截器中验证时，多加一个验证，即前端上报的Token必须与内存或数据库中Token保持一致，否则验证不通过。

```
备注：第一种方式虽然会造成旧的Token可以一直使用，但是有利于分布式部署。另外存在数据库可是也可以，对分布式没有影响（使用同一个数据库的话）。但是当存储在内存当中（比如Session）时，对分布式就不太友好了。
```

主动失效：
		一般管理系统都会有用户注销功能，这时，需要主动让Token失效。这时需要配合内存、数据库实现，即在内存、数据库中保存 一份最新的、有效的Token，当注销时，将存储的Token清空。同时在拦截器验证时，发现内存、数据库的Token是空值时，则验证不通过。

（做个记号，还没写完，需要补充）






































