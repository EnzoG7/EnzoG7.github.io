---
title: Web登录校验之会话技术
date: 2025-07-17 19:59:48
tags:
- Java
- Web
categories:
- JavaWeb
comments: true
---
登录校验,指的是我们在服务器端接收到浏览器发送过来的请求之后,首先我们要对请求进行校验.先要校验一下用户登录了没有,如果用户已经登录了,就直接执行对应的业务操作就可以了;如果用户没有登录,此时就不允许他执行相关的业务操作,直接给前端响应一个错误的结果,最终跳转到登录页面,要求他登录成功之后,再来访问对应的数据.
主要涉及两个技术,会话技术与统一拦截技术.
本文介绍会话技术.

<!-- more -->
# 会话技术
用户打开浏览器就会 建立会话,直到断开连接前产生的多次请求和响应都是在这一次会话中.如果开了多个浏览器就是产生多个会话.

# 会话跟踪技术
为了维护浏览器状态,需要使用会话跟踪技术维护浏览器的状态,以便在同一次会话的多次请求间共享数据.

## Cookie
cookie是客户端会话跟踪技术,它是存储在客户端浏览器的,我们可以在浏览器第一次发起请求来请求服务器的时候,在服务器端设置一个cookie来存储当前登录用户的用户名及ID.然后服务器响应数据会自动的带上cookie并存储到浏览器本地,此后的每一次请求中,客户端都会把该cookie一同携带到服务端.
![cookie流程](Web登录校验之会话技术/cookie.png)
- 优点：HTTP协议中支持的技术(像Set-Cookie 响应头的解析以及 Cookie 请求头数据的携带,都是浏览器自动进行的,是无需我们手动操作的)
- 缺点：
  - 移动端APP(Android、IOS)中无法使用Cookie
  - 不安全,用户可以自己禁用Cookie
  - Cookie不能跨域

## Session
session是服务器端会话跟踪技术,存储在服务器端的.而Session的底层其实就是基于Cookie 来实现.
![Session流程](Web登录校验之会话技术/session.png)
- 优点：Session是存储在服务端的,安全
- 缺点：
  - 服务器集群环境下无法直接使用Session
  - 移动端APP(Android、IOS)中无法使用Cookie
  - 用户可以自己禁用Cookie
  - Cookie不能跨域
Session 底层是基于Cookie实现的会话跟踪,如果Cookie不可用,则该方案,也就失效了.

## Token(Jwt令牌)
如果通过令牌技术来跟踪会话,我们就可以在浏览器发起请求.在请求登录接口的时候,如果登录成功,就可以生成一个令牌,令牌就是用户的合法身份凭证.接下来在响应数据的时候,就可以直接将令牌响应给前端.
接下来我们在前端程序当中接收到令牌之后,就需要将这个令牌存储起来.这个存储可以存储在 cookie 当中,也可以存储在其他的存储空间(比如：localStorage)当中.
接下来,在后续的每一次请求当中,都需要将令牌携带到服务端.携带到服务端之后,接下来我们就需要来校验令牌的有效性.如果令牌是有效的,就说明用户已经执行了登录操作,如果令牌是无效的,就说明用户之前并未执行登录操作.
此时,如果是在同一次会话的多次请求之间,我们想共享数据,我们就可以将共享的数据存储在令牌当中就可以了.
![Token流程](Web登录校验之会话技术/token.png)
- 优点：
  - 支持PC端、移动端
  - 解决集群环境下的认证问题
  - 减轻服务器的存储压力(无需在服务器端存储)
- 缺点：需要自己实现(包括令牌的生成、令牌的传递、令牌的校验)

# Jwt令牌
前面介绍三种会话跟踪技术,上面两种传统的会话技术,在现在的企业开发当中存在很多的问题.为了解决这些问题,在现在的企业开发当中,基本上都会采用第三种方案Jwt令牌技术.
## Jwt的组成
JWT的组成： (JWT令牌由三个部分组成,三个部分之间使用英文的点来分割)
- 第一部分:Header(头), 记录令牌类型、签名算法等. 例如：{"alg":"HS256","type":"JWT"}
- 第二部分:Payload(有效载荷),携带一些自定义信息、默认信息等. 例如：{"id":"1","username":"Tom"}
- 第三部分:Signature(签名),防止Token被篡改、确保安全性.将header、payload,并加入指定秘钥,通过指定签名算法计算而来.
![Jwt组成](Web登录校验之会话技术/jwt组成.png)

## Jwt编码原理
- 对JSON格式的数据进行一次base64编码.
- Base64：是一种基于64个可打印的字符来表示二进制数据的编码方式.既然能编码,那也就意味着也能解码.所使用的64个字符分别是A到Z、a到z、 0- 9,一个加号,一个斜杠,加起来就是64个字符.任何数据经过base64编码之后,最终就会通过这64个字符来表示.当然还有一个符号,那就是等号.等号它是一个补位的符号.
- 需要注意的是Base64是编码方式,而不是加密方式.

## Jwt Maven依赖
```xml
<!-- JWT依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

## Jwt生成代码实现
```java
@Test
public void testGenJwt() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", 10);
    claims.put("username", "itheima");

    String jwt = Jwts.builder().signWith(SignatureAlgorithm.HS256, "aXRjYXN0")
        .addClaims(claims)
        .setExpiration(new Date(System.currentTimeMillis() + 12 * 3600 * 1000))
        .compact();

    System.out.println(jwt);
}
```

## Jwt校验代码实现
```java
@Test
public void testParseJwt() {
    Claims claims = Jwts.parser().setSigningKey("aXRjYXN0")
        .parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MTAsInVzZXJuYW1lIjoiaXRoZWltYSIsImV4cCI6MTcwMTkwOTAxNX0.N-MD6DmoeIIY5lB5z73UFLN9u7veppx1K5_N_jS9Yko")
        .getBody();
    System.out.println(claims);
}
```