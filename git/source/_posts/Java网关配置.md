---
title: Java网关配置
date: 2025-09-10 20:32:36
tags: 
- SpringBoot
- Java
- 网关
- 微服务
categories:
- 微服务
comments: true
---
前端请求网关根据请求路径路由到微服务,网关从nacos获取微服务实例地址将请求转发到具体的微服务实例上.
现在要根据需求使用Java在网关实现路由转发和用户身份认证的功能：
- 根据请求Url路由到具体的微服务
- 校验用户的token,取出token中的用户信息
- 从nacos中取出服务实例进行负载均衡
所以使用java开发的网关,如
- Spring Cloud Gateway:基于Spring的WebFlux技术,完全支持响应式编程,吞吐能力更强
- NetFlix Zuul:早期实现,已淘汰
![网关架构图](java网关配置/网关逻辑.png)

<!--more-->

# 实现网关路由
## 添加依赖
在网关工程的`pom.xml`中添加依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>xxx-parent</artifactId>
        <groupId>com.xxx</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>xxx-gateway</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.xxx</groupId>
            <artifactId>xxx-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--nacos discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 新建启动类
```java
package com.xxx.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

## 配置路由
在`application.yml`中配置路由
```yaml
server:
  port: 8080
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: 虚拟机IP:8848
    gateway:
      routes:
        - id: aaa
          uri: lb://aaa-service
          predicates:
            - Path=/aaas/**
        - id: bbb
          uri: lb://bbb-service
          predicates:
            - Path=/bbbs/**
        - id: user
          uri: lb://user-service
          predicates:
            - Path=/users/**,/addresses/**
```
路由规则routes包括四个属性,定义语法如下：
- id：路由的唯一标示
- predicates：路由断言,`Predicates`是用于判断请求是否满足特定条件的组件
- filters：路由过滤条件
- uri：路由目标地址,lb://代表负载均衡,从注册中心获取目标微服务的实例列表,并且负载均衡选择一个访问

`predicates路由断言`的类型:
![路由断言](java网关配置/路由断言.png)

# 网关鉴权
把身份校验的工作放到网关,确保只有经过授权的用户或设备才能访问特定的服务或资源：
- 在网关和用户服务保存秘钥
- 在网关开发身份校验功能
![网关鉴权](java网关配置/网关鉴权.png)
流程如下:
- 用户登录成功生成token并存储在前端
- 前端携带token访问网关
- 网关解析token中的用户信息,网关将请求转发到微服务,转发时携带用户信息
- 微服务从http头信息获取用户信息
- 微服务之间远程调用使用内部接口（无状态接口）

## 网关内置过滤器
内置过滤器有很多，具体在工作中根据需求去使用
![内置过滤器](java网关配置/内置过滤器.png)
例如使用StripPrefix过滤器的网关的路由配置如下
```yaml
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
  filters:
    - StripPrefix=1
```
StripPrefix=1表示去除一级路径前缀,使用StripPrefix=1后
请求：http://localhost:8080/product/items/page?pageNo=1&pageSize=1
路径到：http://localhost:8081/items/page?pageNo=1&pageSize=1(8081是目标微服务的端口)

## 自定义过滤器
无论是GatewayFilter还是GlobalFilter都支持自定义,只不过编码方式、使用方式略有差别
### 自定义GlobalFilter
全局过滤器不用在路由中配置
```java
@Component
@Slf4j
public class PrintAnyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 编写过滤器逻辑
        log.info("打印全局过滤器");
        // 放行
         return chain.filter(exchange);

        // 拦截
//        ServerHttpResponse response = exchange.getResponse();
//        response.setRawStatusCode(401);
//        return response.setComplete();
    }

    @Override
    public int getOrder() {
        // 过滤器执行顺序，值越小，优先级越高
        return 0;
    }
}
```

### 自定义GatewayFilter
自定义GatewayFilter不是直接实现GatewayFilter,而是继承AbstractGatewayFilterFactory,`该类的名称一定要以GatewayFilterFactory为后缀`
```java
@Component
@Slf4j
public class FirstFilterGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {

    @Override
    public GatewayFilter apply(Object config) {
        return new GatewayFilter() {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                ServerHttpRequest request = exchange.getRequest();

                log.info("请求路径：{}",request.getPath());
                log.info("网关过滤器FirstFilterGatewayFilterFactory执行啦...");
                //放行
                return chain.filter(exchange);
                //拦截 返回401状态码
                //exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                //return exchange.getResponse().setComplete();
            }
        };
    }
}
```
配置仅在product路由中有效的路由
```yaml
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
  filters:
    - StripPrefix=1
    - FirstFilter # 此处直接以自定义的GatewayFilterFactory类名称前缀类声明过滤器
```
配置在所有路由中都有效的路由
```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - FirstFilter # 此处直接以自定义的GatewayFilterFactory类名称前缀类声明过滤器
```

## 身份校验过滤器
利用自定义`GlobalFilter`来完成身份校验,JWT工具类准备好,如:
- AuthProperties：配置身份校验需要拦截的路径,因为不是所有的路径都需要登录才能访问
- JwtProperties：定义与JWT工具有关的属性,比如秘钥文件位置
- SecurityConfig：工具的自动装配
- JwtTool：JWT工具,其中包含了校验和解析token的功能
- hmall.jks：秘钥文件

### 配置白名单
其中`AuthProperties`和`JwtProperties`所需的属性要在application.yaml中配置
```ymal
hm:
  jwt:
    location: classpath:hmall.jks # 秘钥地址
    alias: hmall # 秘钥别名
    password: hmall123 # 秘钥文件密码
    tokenTTL: 30m # 登录有效期
  auth:
    excludePaths: # 无需身份校验的路径
      - /search/**
      - /users/login
      - /items/**
```
### 定义过滤器
```java
@Component
@RequiredArgsConstructor
@EnableConfigurationProperties(AuthProperties.class)
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    private final JwtTool jwtTool;

    private final AuthProperties authProperties;

    private final AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取Request
        ServerHttpRequest request = exchange.getRequest();
        //请求路径
        String path = request.getPath().toString();
        //白名单
        List<String> excludePaths = authProperties.getExcludePaths();
        //判断当前请求路径是否是白名单,使用antPathMatcher进行匹配
        for (String excludePath : excludePaths) {
            boolean match = antPathMatcher.match(excludePath, path);
            if (match) {
                return chain.filter(exchange);
            }
        }
        //获取请求头的token
        String token = exchange.getRequest().getHeaders().getFirst("authorization");
        if(token==null){
            //如果token为空则返回401错误
            ServerHttpResponse response = exchange.getResponse();
            response.setRawStatusCode(401);
            return response.setComplete();
        }
        //如果token不为空则校验token
        Long userId = null;
        try {
            userId = jwtTool.parseToken(token);
        } catch (Exception e) {
            //返回token无效的错误
            ServerHttpResponse response = exchange.getResponse();
            response.setRawStatusCode(401);
            return response.setComplete();
        }

        // 5.如果有效，传递用户信息
       exchange.getRequest().mutate().header("user-info",userId.toString());
       
        // 6.放行
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

## 网关传递用户信息到微服务
![传递用户信息](java网关配置/传递用户信息.png)
利用SpringMVC的拦截器来实现登录用户信息获取,并存入ThreadLocal:
- 改造网关过滤器,在获取用户信息后保存到请求头,转发到下游微服务
- 编写微服务拦截器,拦截请求获取用户信息,保存到ThreadLocal后放行

### 改造过滤器
身份校验拦截器的处理逻辑，保存用户信息到请求头中
```java
exchange.getRequest().mutate().header("user-info",userId.toString());
```

ThreadLocal工具返回从threadLocal中获取的userId
```java
public static Long getUser() {
    return tl.get();
}
```

### 编写拦截器
编写拦截器,获取用户信息并保存到UserContext,然后放行
拦截器我们直接写在xxx-common中,并写好自动装配.这样微服务只需要引入xxx-common就可以直接具备拦截器功能
```java
package com.xxx.common.interceptor;
public class UserInfoInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1.获取请求头中的用户信息
        String userInfo = request.getHeader("user-info");
        // 2.判断是否为空
        if (StrUtil.isNotBlank(userInfo)) {
            // 不为空，保存到ThreadLocal
                UserContext.setUser(Long.valueOf(userInfo));
        }
        // 3.放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户
        UserContext.removeUser();
    }
}
```

### 配置拦截器
接着在xxx-common模块下编写SpringMVC的配置类,配置登录拦截器
```java
package com.xxx.common.config;
@Configuration
@ConditionalOnClass(DispatcherServlet.class)
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new UserInfoInterceptor());
    }
}
```

需要注意的是,这个配置类默认是不会生效的,因为它所在的包是com.xxx.common.config,与其它微服务的扫描包不一致,无法被扫描到,因此无法生效。
基于SpringBoot的自动装配原理,我们要将其添加到`resources目录`下的`META-INF/spring.factories`文件中
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hmall.common.config.MyBatisConfig,\
  com.hmall.common.config.JsonConfig,\
  com.hmall.common.config.MvcConfig
```

## Feign接口传递用户
有些业务是比较复杂的,请求到达微服务后还需要调用其它多个微服务,并没有传递用户信息

### 方案1：OpenFeign拦截器
微服务之间调用是基于OpenFeign来实现的,在进行OpenFeign调用时可以将用户信息放在http头中传递
借助实现Feign中提供的一个拦截器接口：feign.RequestInterceptor可以实现Feign拦截器
```java

public class FeignInterceptorConfig  {
    @Bean
    public RequestInterceptor userInfoRequestInterceptor(){
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                // 获取登录用户
                Long userId = UserContext.getUser();
                if(userId == null) {
                    // 如果为空则直接跳过
                    return;
                }
                // 如果不为空则放入请求头中，传递给下游微服务
                template.header("user-info", userId.toString());
            }
        };
    }
}
```
方案1就是在feign远程调用前在http头中添加用户信息，请求到达微服务由微服务拦截器解析出http头中的user-info放入ThreadLocal

### 方案2: 单独编写对应接口