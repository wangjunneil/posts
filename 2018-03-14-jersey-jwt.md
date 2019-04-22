---
title: 使用jersey实现jwt认证
name: jersey-jwt
date: 2018-03-14
tags: [jersey, jwt]
categories: [Java]
---

* 目录
{:toc}

---

使用maven创建一个普通的webapp工程，restful方面使用实现了JAX-RS的**jersey**库，jwt授权实现使用**jjwt**([github.com/jwtk/jjwt](//github.com/jwtk/jjwt))。

+ **jersey**，[jersey.github.io/](//jersey.github.io)
+ **jwt**，[jwt.io/](//jwt.io)

## 1. 授权流程

![jwt](//vinnycc.oss-cn-shanghai.aliyuncs.com/20190322/JWT_tokens_EN.png)

## 2. 实现方式

### 2.1 使用maven创建webapp工程

```shell
$ mvn archetype:generate
```

### 2.2 maven的pom.xml添加相关依赖

```xml
<dependency> <!-- jersey -->
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet</artifactId>
    <version>2.17</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>2.17</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.17</version>
</dependency>

<dependency> <!-- jwt -->
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

### 2.3 在web.xml中配置jersey

```xml
<servlet>
    <servlet-name>jersey-restful</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>jersey.config.server.provider.packages</param-name>
        <param-value>cc.vinny.jersey</param-value> <!-- restful实现的package -->
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>jersey-restful</servlet-name>
    <url-pattern>/rest/*</url-pattern> <!-- restful的rootpath -->
</servlet-mapping>
```

### 2.4 编写获取token的rest服务

`AuthenticationResource.java` 是一个标准的rest实现类，其中`POST /login`方法根据输入的用户名、密码生成授权jwt的token值。

```java
package cc.vinny.jersey;

import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.util.Date;

@Path("authentication")
public class AuthenticationResource {

    @POST
    @Path("/login")
    @Produces(MediaType.APPLICATION_JSON)
    public Response authLogin(User user) {
        String username = user.getUsername();
        String password = user.getPassword();

        // TODO 进行授权登录验证操作，这里做示例
        if ("admin".equals(username) && "admin".equals(password)) {
            long currentTimeMillis = System.currentTimeMillis();
            Date now = new Date(currentTimeMillis);
            Date expireTime = new Date(currentTimeMillis + 86400000);   // 失效时间24小时后
            System.out.println(now + " ~ " + expireTime);

            String jwt = Jwts.builder()
                    .setId("1") // 设置版本
                    .setAudience("user")    // 设置角色
                    .signWith(SignatureAlgorithm.HS256, "THIS_SECURITY_KEY")    // 设置加密类型及密钥
                    .setSubject(username)   // 设置标题
                    .setIssuedAt(now)       // 设置签发时间
                    .setExpiration(expireTime)  // 设置过期时间
                    .claim("email", "abc@gmail.com")    // 可以自定义属性
                    .compact();

            return Response.status(Response.Status.CREATED).entity("{\"jwt\":\"" + jwt + "\"}").build();
        }

        return Response.status(Response.Status.UNAUTHORIZED).build();
    }
}
```

客户端进行鉴权操作，得到jwt的token值

```shell
$ curl -X POST -H "Content-Type: application/json" //localhost:8080/web/rest/authentication/login -d '
{
        "username" : "admin",
        "password" : "admin"
}'
```

### 2.5 编写拦截器对业务资源进行授权拦截

`ResourceFilter.java`对所有请求进行授权拦截，在请求头中是否包含`Authorization`的验证，以判断该请求的合法性。

```java
package cc.vinny.jersey;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.Jwts;
import org.glassfish.jersey.server.ContainerRequest;

import javax.annotation.Priority;
import javax.ws.rs.Priorities;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.core.Response;
import javax.ws.rs.ext.Provider;

@Provider
@Priority(Priorities.AUTHENTICATION)    // 过滤器的最高优先级
public class ResourceFilter implements ContainerRequestFilter {

    public void filter(ContainerRequestContext ctx) {
        String method = ctx.getMethod().toLowerCase();
        String path = ((ContainerRequest) ctx).getPath(true).toLowerCase();
        // 过滤非验证资源，如授权登录、注册等静态资源
        if ("post".equals(method) && "authentication/login".equals(path)) {
            // ctx.setSecurityContext(new SecurityContextAuthorizer(uriInfo,new AuthorPricinple(name), new String[]{"user"}));
            return;
        }

        Response response = null;
        final String authHeader = ctx.getHeaderString("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            response = Response.status(Response.Status.UNAUTHORIZED)
                    .entity("Missing or invalid Authorization header").build();
        } else {
            final String token = authHeader.substring(7);
            try {
                Claims jwts = Jwts.parser()
                        .setSigningKey("THIS_SECURITY_KEY")
                        .parseClaimsJws(token)
                        .getBody();

                // 根据username查询是否存在此用户
                // String username = jwts.getSubject();
                // 判断token版本是否一致
                // String version = jwts.getId();
            } catch (ExpiredJwtException e) {
                response = Response.status(Response.Status.FORBIDDEN).entity("Token is expired").build();
            } catch (Exception e) {
                response = Response.status(Response.Status.FORBIDDEN).entity("Token is invalid").build();
            }
        }

        ctx.abortWith(response);
    }
}
```

### 2.6 业务资源的rest实现

`UserResource.java`类用于针对普通资源的CRUD操作，对于此类请求，会被`ResourceFilter.java`拦截，只有jwt的token符合要求才允许客户端访问资源。

```java
package cc.vinny.jersey;

import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;
import java.util.Arrays;
import java.util.List;

@Path("user")
public class UserResource {

    @GET
    @Path("/list")
    @Produces(MediaType.APPLICATION_JSON)
    public List<User> listUsers() {
        User [] users = new User[] {
            new User(1, "john", "123321"),
            new User(2, "smith", "000000")
        };
        return Arrays.asList(users);
    }

    @DELETE
    @Path("/{userId}")
    @Produces(MediaType.APPLICATION_JSON)
    public String deleteUser(@PathParam("userId") String userId) { //localhost:8080/web/rest/user/1
        System.out.println(userId);
        return "{\"status\":0}";
    }

    @GET
    @Path("/{userId}")
    @Produces(MediaType.APPLICATION_JSON)
    public User getUser(@PathParam("userId") int userId) { //localhost:8080/web/rest/user/1
        return new User(userId, "john", "123321");
    }

    @POST
    @Path("/create")
    @Produces(MediaType.APPLICATION_JSON)
    public User createUser(User user) {
        return new User(user.getUserId(), user.getUsername(), user.getPassword());
    }

    @PUT
    @Path("/update")
    @Produces(MediaType.APPLICATION_JSON)
    public User updateUser(User user) {
        return new User(user.getUserId(), user.getUsername(), user.getPassword());
    }
}
```

下面是模拟客户端对业务资源请求的访问

```shell
$ curl -X GET -H "Authorization: Bearer xxx.xxx.xxxxxxx" //localhost:8080/web/rest/user/list

$ curl -X DELETE -H "Authorization: Bearer xxx.xxx.xxxxxxx" //localhost:8080/web/rest/user/1

$ curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer v" //localhost:8080/web/rest/user/create -d '
{
"userId" : 3,
"username" : "wangjun",
"password" : "123321"
}'
```
