---
title: jwt
date: 2020-01-15 10:52:16
categories: spring
tags: jwt

---

### 概述

JSON web Token（jwt）是一种跨域认证解决方案

#### 原理

JWT的原理是，服务器认证以后，生成一个JSON对象，发回给用户，以后再次与服务端通信时，只需要发回这个JSON对象，服务端通过JSON对象识别用户身份，同时为了防止对象数据被篡改，生成JSON时会加上签名

#### 组成

JWT由三个部分组成，分别为Header、Payload、Signature

具体见[此文](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

**头部（Header）**

头部用于描述关于该JWT的最基本的信息，如其类型及使用的签名算法，这也可以表示成一个JSON对象

`{"typ":"JWT","alg":"HS256"}`

在头部指明了签名算法是HS256算法，用BASE64进行编码后得到的字符串：`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9`

**载荷（payload）**

载荷就是存放有效信息的地方，包括三类信息：

- 标准注册中的声明（建议但不强制使用）

- 公共的声明

- 私有的声明

  指自定义的claim，如用户名和密码。

定义一个payload:

`{"sub":"1234567890","name":"John Doe","admin":true}`，使用BASE64进行加密，得到JWT的第二部分

`eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9`

**签名（signature)**

JWT的第三部分是一个签名信息，签名信息由三部分组成：

1. header（base64加密后）
2. payload（base64加密后）
3. 私钥（保存在服务器）

使用头部定义的签名算法对以上三部分进行计算得出第三部分的签名

最后把三个部拼成一个字符串，用`"."`分隔，返回给用户

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

### SpringBoot集成JWT

[参考原文](https://www.jianshu.com/p/99a458c62aa4)

1. pom.xml添加maven依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/com.auth0/java-jwt -->
   <dependency>
       <groupId>com.auth0</groupId>
       <artifactId>java-jwt</artifactId>
       <version>3.8.1</version>
   </dependency>
   ```

2. 实现签名和认证方法

   ```java
   package ***.config;
   
   import ***.entity.Role;
   import ***.entity.User;
   import ***.repo.UserRepo;
   import io.jsonwebtoken.*;
   import org.apache.tomcat.util.codec.binary.Base64;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.security.crypto.password.PasswordEncoder;
   
   import javax.crypto.SecretKey;
   import javax.crypto.spec.SecretKeySpec;
   import java.util.*;
   import java.util.stream.Collectors;
   
   public class JwtTokenWrapper {
       @Value("${constant.exp_millis}")
       private int expMillis;
       @Value("${constant.fresh_millis}")
       private int freshMillis;
       private String issuer = "hlwb";
       private JwtParser jwtParser = Jwts.parser()
               .requireIssuer(issuer)
               .setSigningKey(generalKey());
       private static final JwtTokenWrapper INSTANCE = new JwtTokenWrapper();
   
       public static JwtTokenWrapper getInstance() {
           return INSTANCE;
       }
       private JwtTokenWrapper() {
       }
   
       /**
        * 由字符串生成加密key
        * @return 加密key
        */
       private SecretKey generalKey(){
           String stringKey = "Jx7CHGa4ew#d@n69bWUoGwbO1@n1BnaCk7G2^0i97fCk*Ae3#vs#uIpIPyF4ItzX";
           byte[] encodedKey = Base64.decodeBase64(stringKey);
           return new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
       }
   
       /**
        * 创建一个包含用户Id和用户名的 token
        * @param userDetails user对象
        * @return token
        */
       public String createToken(User userDetails) {
           Date now = new Date();
           Date expAt;
           if (expMillis == 0) {
               expAt = null;
           } else {
               long nowMillis = now.getTime();
               long exp = nowMillis + expMillis;
               expAt = new Date(exp);
           }
           JwtBuilder builder = Jwts.builder()
                   .setClaims(new HashMap<String, Object>(2) {{
                       this.put("username", userDetails.getUserName());
                       this.put("id", userDetails.getId());
                       this.put("authorities", userDetails.getRoleSet()
                               .stream()
                               .map(Role::getName)
                               .collect(Collectors.toSet()));
                   }}).setIssuer(issuer)
                   .setId(Long.toString(userDetails.getId()))
                   .setIssuedAt(now)
                   .setExpiration(expAt)
                   .signWith(SignatureAlgorithm.HS256, generalKey());
           return builder.compact();
       }
   
       /**
        * claim，可以称载荷，playload
        * parser是解析器
        * @param token 令牌
        * @return body
        */
       public Claims getClaims(String token) {
           try {
               return jwtParser.parseClaimsJws(token)
                       .getBody();
           } catch (Exception e) {
               return null;
           }
       }
   
       /**
        * 转换成User对象
        * @param claims claims
        * @return user
        */
       @SuppressWarnings("unchecked")
       public User parseUser(Claims claims) {
           String username = (String) claims.get("username");
           long id = Long.parseLong(claims.get("id").toString());
           Set<Role> roleSet = ((List<String>) claims.get("authorities"))
                   .stream()
                   .map(Role::new)
                   .collect(Collectors.toSet());
           User user = new User();
           user.setUserName(username);
           user.setId(id);
           user.setRoleSet(roleSet);
           return user;
       }
   
       /**
        * 检查 token 是否需要刷新
        * @param claims claims
        * @return true 需要刷新,false 不需要刷新
        */
       public boolean timerefreshToken(Claims claims){
           if (freshMillis == 0) {
               return false;
           } else {
               Date exp = claims.getExpiration();
               return exp.before(new Date(System.currentTimeMillis() + expMillis - freshMillis));
           }
       }
   
       public static void main(String[] args) {
       }
   }
   ```

3. 添加拦截器

   ```java
   package ***
   
   import ***.entity.Role;
   import ***.entity.User;
   import ***.repo.UserRepo;
   import ***.service.ResourceService;
   import io.jsonwebtoken.Claims;
   import org.apache.logging.log4j.LogManager;
   import org.apache.logging.log4j.Logger;
   import org.springframework.util.Assert;
   import org.springframework.web.servlet.HandlerInterceptor;
   import org.springframework.web.servlet.ModelAndView;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.util.HashSet;
   import java.util.Map;
   import java.util.Set;
   import java.util.stream.Collectors;
   
   
   /**
    * JWT过滤器
    */
   public class JwtInterceptor implements HandlerInterceptor {
       private static final Logger log = LogManager.getLogger();
   
       private final JwtTokenWrapper jtw = JwtTokenWrapper.getInstance();
       private final ResourceService resourceService;
       private final Set<String> adminset = new HashSet<>(1);
   
       public JwtInterceptor(ResourceService resourceService) {
           this.resourceService = resourceService;
           this.adminset.add("ROLE_ADMIN");
       }
   
       @Override
       public boolean preHandle(HttpServletRequest httpServletRequest,
                                HttpServletResponse httpServletResponse,
                                Object o) throws Exception {
           // 不拦截OPTIONS方法的请求
           String method = httpServletRequest.getMethod();
           if ("OPTIONS".equals(method)) {
               return true;
           }
   
           String url = httpServletRequest.getRequestURI();
           try {
               // 提取header中的token
               String token = httpServletRequest.getHeader("Authorization");
               Assert.notNull(token, "headerAuthorizationNotFound");
   
               Claims claims = jtw.getClaims(token);
               // token是否有效
               Assert.notNull(claims, "token invalid");
               User user = jtw.parseUser(claims);
               Set<Role> gac = user.getRoleSet();
               Assert.notNull(gac, "access denied, UserID:" + user.getId());
               // 验证url权限
               if (url.startsWith("/admin")) {
                   // 处理admin权限
                   Assert.isTrue(isAuthed(adminset, gac.stream()
                           .map(Role::getName)
                           .collect(Collectors.toSet())), "access denied, UserID:" + user.getId());
                   log.info("token valid, UserID:{}, pass url:{}", user.getId(), url);
                   return true;
               } else {
                   // 处理非admin权限
                   Map<String, Set<String>> resourceMap = resourceService.loadResourceRoleMap();
                   Set<String> cac = resourceMap.getOrDefault(url, adminset);
                   cac.addAll(adminset);
                   Assert.isTrue(isAuthed(cac, gac.stream()
                           .map(Role::getName)
                           .collect(Collectors.toSet())), "access denied, UserID:" + user.getId());
                   log.info("token valid, UserID:{}, pass url:{}", user.getId(), url);
                   return true;
               }
           } catch (IllegalArgumentException e) {
               log.warn("{}, filter url:{}", e.getMessage(), url);
               httpServletResponse.sendError(401);
               return false;
           }
       }
   
       @Override
       public void postHandle(HttpServletRequest request,
                              HttpServletResponse response, Object handler,
                              ModelAndView modelAndView) {
       }
   
       @Override
       public void afterCompletion(HttpServletRequest request,
                                   HttpServletResponse response, Object handler, Exception ex) {
       }
   
       /* gac 为用户所被赋予的权限。 cac 为访问相应的资源应该具有的权限。 */
       private boolean isAuthed(Set<String> cac, Set<String> gac) {
           if (cac.size() > gac.size()) { ;
               for (String ga : gac) {
                   if (cac.contains(ga)) {
                       return true;
                   }
               }
               return false;
           } else {
               for (String ca : cac) {
                   if (gac.contains(ca)) {
                       return true;
                   }
               }
               return false;
           }
       }
   
   }
   
   ```

4. 将拦截器注册进容器

   ```java
   package ***.config;
   
   import ***
   
   @Configuration
   public class WebMvcConfig extends WebMvcConfigurerAdapter {
       private final ResourceService resourceService;
   
       @Autowired
       public WebMvcConfig(ResourceService resourceService) {
           this.resourceService = resourceService;
       }
   
       /**
        * JWT验证
        */
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(new JwtInterceptor(resourceService))
                   .addPathPatterns("/**")
                   .excludePathPatterns("/auth/**")
                   .excludePathPatterns("/callBack/**")
                   .excludePathPatterns("/error")
                   .excludePathPatterns("/swagger-ui.html")
                   .excludePathPatterns("/webjars/**")
                   .excludePathPatterns("/v2/api-docs")
                   //.excludePathPatterns("/admin/signup")
                   .excludePathPatterns("/swagger-resources/**");
       }
   
   }
   
   ```

   

### 补充：

网上看到的jwt工具：[原文](https://www.jianshu.com/p/aaf23f42abb3)

#### 第一种

依赖:

```xml
<dependency>
         <groupId>com.auth0</groupId>
          <artifactId>java-jwt</artifactId>
          <version>3.4.0</version>
 </dependency>
```

工具类：

```java
public class JavaJWTUtil {
    // 过期时间5分钟
    private static final long EXPIRE_TIME = 5 * 60 * 1000;

    /**
     * 生成签名,5min后过期
     *
     * @param username 用户名
     * @param secret   用户的密码
     * @return 加密的token
     */
    public static String sign(String username, String secret) {
        Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
        Algorithm algorithm = Algorithm.HMAC256(secret);
        // 附带username信息
        return JWT.create()
                .withClaim("username", username)
                .withClaim("as", "a")
                .withExpiresAt(date)
                .sign(algorithm);
    }
	/**
	*对token进行认证
	*/
    public static boolean verify(String token, String username, String secret) {
        try {
            Algorithm algorithm = Algorithm.HMAC256(secret);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withClaim("username", username)
                    .build();
            DecodedJWT jwt = verifier.verify(token);
            System.out.println(token);
            System.out.println(jwt.getHeader());
            System.out.println(jwt.getPayload());
            System.out.println(jwt.getSignature());
            System.out.println(jwt.getToken());
            return true;
        } catch (Exception exception) {
            exception.printStackTrace();
            return false;
        }
    }
    public static void main(String[] args) {
        String zhangsan = JavaJWTUtil.sign("zhangsan", "123");
        JavaJWTUtil.verify(zhangsan, "zhangsan", "123");
    }
}
```

#### 第二种：

依赖：

```xml
 <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```

工具类：

```java
public class JJWTUtil {
    public static String createJWT(String id, String subject, long ttlMillis) throws Exception {
        //指定签名的时候使用的签名算法，也就是header那部分，jjwt已经将这部分内容封装好了。
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        //生成JWT的时间
        long nowMillis = System.currentTimeMillis();
        //创建payload的私有声明（根据特定的业务需要添加，如果要拿这个做验证，一般是需要和jwt的接收方提前沟通好验证方式的）
        Date now = new Date(nowMillis);
        Map<String, Object> claims = new HashMap<>();
        claims.put("uid", "DSSFAWDWADAS...");
        claims.put("user_name", "admin");
        claims.put("nick_name", "DASDA121");
        //生成签名的时候使用的秘钥secret,这个方法本地封装了的，一般可以从本地配置文件中读取，切记这个秘钥不能外露哦。
        // 它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
        SecretKey key = generalKey();
        //下面就是在为payload添加各种标准声明和私有声明了
        JwtBuilder builder = Jwts.builder() //这里其实就是new一个JwtBuilder，设置jwt的body
                .setClaims(claims)          //如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
                .setId(id)                  //设置jti(JWT ID)：是JWT的唯一标识，根据业务需要，这个可以设置为一个不重复的值，主要用来作为一次性token,从而回避重放攻击。
                .setIssuedAt(now)           //iat: jwt的签发时间
                .setSubject("hehehhe")        //sub(Subject)：代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志。
                .signWith(signatureAlgorithm, key);//设置签名使用的签名算法和签名使用的秘钥
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date exp = new Date(expMillis);
            builder.setExpiration(exp);     //设置过期时间
        }
        return builder.compact();           //就开始压缩为xxxxxxxxxxxxxx.xxxxxxxxxxxxxxx.xxxxxxxxxxxxx这样的jwt
    }

    public static SecretKey generalKey() {
        String stringKey = "21341234243134";//本地配置文件中加密的密文7786df7fc3a34e26a61c034d5ec8245d
        byte[] encodedKey = Base64.decodeBase64(stringKey);//本地的密码解码[B@152f6e2
        // 根据给定的字节数组使用AES加密算法构造一个密钥，使用 encodedKey中的始于且包含 0 到前 leng 个字节这是当然是所有
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    
	//解析jwt得到claims
    public static Claims parseJWT(String jwt) throws Exception {
        try {
            SecretKey key = generalKey();  //签名秘钥，和生成的签名的秘钥一模一样
            Claims claims = Jwts.parser()  //得到DefaultJwtParser
                    .setSigningKey(key)         //设置签名的秘钥
                    .parseClaimsJws(jwt).getBody();//设置需要解析的jwt
            return claims;
        } catch (SignatureException | MalformedJwtException e) {
            throw e;
        } catch (ExpiredJwtException e) {
            // jwt 已经过期，在设置jwt的时候如果设置了过期时间，这里会自动判断jwt是否已经过期，如果过期则会抛出这个异常，我们可以抓住这个异常并作相关处理。
        }
        return null;
    }

    public static void main(String[] args) throws Exception {
        String jwt = JJWTUtil.createJWT("", "", new Date().getTime());
        System.out.println(jwt);
        System.out.println(JJWTUtil.parseJWT(jwt));
    }

}
```