---
title: OpenFeign配置
date: 2025-09-10 20:01:40
tags: 
- SpringBoot
- Java
- OpenFeign
- 微服务
categories:
- 微服务
comments: true
---
OpenFeign技术可以让远程调用像本地方法调用一样简单,OpenFeign是一个声明式的HTTP客户端框架,它简化了编写 REST 客户端的过程

<!--more-->
# 添加依赖
在项目中添加OpenFeign的依赖,最好就在抽取的api工程中
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

# 创建Feign客户端
- 在api工程定义一个接口来描述远程服务。
- 使用`@FeignClient`注解来标识远程调用接口,并指定服务名、接口路径.
- 对应服务工程引入api工程
```xml
<dependency>
    <groupId>com.hmall</groupId>
    <artifactId>hm-api</artifactId>
    <version>1.0.0</version>
</dependency>
```
- 对应服务工程的`Controller`实现该`Feign`接口
```java
@FeignClient(name = "xxx-service")
public interface XxxClient {
    @GetMapping("/xxxx/{id}")
    Xxx getXxx(@PathVariable("id") String id);

    @PostMapping("/xxxx")
    Xxx createXxx(@RequestBody Xxx xxx);

    // 其他方法.
    ......
}
```
接口中的几个关键信息:
- `@FeignClient(name="xxx-service")` ：声明服务名称
- `@FeignClient(path = "/xxxx")`: 相当于XxxController类上的@RequestMapping("/xxxx")指定的路径
- `@GetMapping` ：声明请求方式
- `@RequestParam("ids") Collection<Long> ids` ：声明请求参数
- `List<XxxDTO>` ：返回值类型

# 使用Feign客户端
在调用者工程的实现类调用`XxxClient`的方法
```java
@Service
public class XxxServiceImpl {
    @Autowired
    private XxxClient xxxClient;

    public void Handle(String id) {
        // 调用XxxCleint
        List<XxxDTO> xxxDTOList = xxxClient.getXxx(id);
    }
}
```

# 启用OpenFeign
在调用者工程的启动类添加`@EnableFeignClients`注解,启用OpenFeign
```java
@SpringBootApplication
@EnableFeignClients
@MapperScan("com.xxx.mapper")
public class XxxApplication {
    public static void main(String[] args) {
        SpringApplication.run(XxxApplication.class, args);
    }
}
```

# OKHttp
Feign底层发起http请求,依赖于第三方的http客户端框架.常用的http客户端有：
- HttpURLConnection：默认实现,不支持连接池
- Apache HttpClient ：支持连接池
- OKHttp：支持连接池

## 更换为OKHttp
在调用者工程添加OKHttp的依赖
```xml
<!--OK http 的依赖 -->
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-okhttp</artifactId>
</dependency>
```
在调用者工程的`application.yml`中添加配置开启Feign的连接池功能
```yaml
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```