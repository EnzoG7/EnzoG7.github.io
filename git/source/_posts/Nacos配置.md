---
title: Nacos配置
date: 2025-09-10 19:43:38
tags: 
- SpringBoot
- Java
- Nacos
- 微服务
categories:
- 微服务
comments: true
---
Nacos是国内产品,中文文档比较丰富,而且同时具备配置管理、服务注册与发现功能

<!-- more -->

# Docker配置
## 拉取Nacos镜像
`nacosxxx.tar`是nacos的镜像文件,nacos文件夹下是nacos配置文件
其中的`nacos/custom.env`文件中,有一个`MYSQL_SERVICE_HOST`也就是mysql地址,需要修改为虚拟机IP地址
```bash
docker load -i nacosxxx.tar
```
## 运行Nacos容器
```bash
docker run -d \
--name nacos \
--env-file ./nacos/custom.env \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--restart=always \
nacos/nacos-server:vxxx
```
## 访问Nacos
浏览器访问`http://虚拟机IP:8848/nacos/`,默认账号密码为`nacos/nacos`

# 服务注册
## 添加依赖
在父工程需要添加Spring Cloud及Spring Cloud Alibaba的版本约束
```xml
<!--spring cloud-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <!--   type=pom ,scope=import 表示继承 和 parent 作用一样             -->
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--spring cloud alibaba-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>${spring-cloud-alibaba.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```
并定义nacos-client的版本
```xml
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>2.4.0</version>
</dependency>
```
在服务工程的`pom.xml`中添加nacos-client的依赖
```xml
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 配置nacos
在服务工程的`application.yml`中添加nacos的配置
```yaml
spring:
  application:
    name: xxxx-service # 服务名称
  cloud:
    nacos:
      server-addr: 虚拟机IP:8848 # nacos地址
```

## 启动服务
显示以下日志为注册成功
```bash
Success to connect to server [192.168.101.68:8848] on start up, connectionId ...
```

# 服务发现
## 添加依赖
由于还需要负载均衡,在服务工程的`pom.xml`中添加SpringCloud提供的LoadBalancer依赖
```xml
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--负载均衡器-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

## 配置nacos地址
在服务工程的`application.yml`中添加nacos的配置
```yaml
spring:
  cloud:
    nacos:
      server-addr: 虚拟机IP:8848
```

## 调用其他服务
任何一个微服务都可以调用别人,也可以被别人调用,即可以是调用者,也可以是提供者
### 方法一
见`OpenFeign`篇,后续方法一个比一个麻烦

### 方法二
调用者工程Config添加`RemoteCallConfig`类,使用`@LoadBalanced`注解标识`RestTemplate`
```java
@Configuration
public class RemoteCallConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();  
    }
}
```
调用者工程的Service使用`RestTemplate`调用提供者
```java
@Service
public class RemoteCallService {
    @Autowired
    private RestTemplate restTemplate;

    public String call() {
        // xxxx-service是提供者的服务名
        return restTemplate.exchange("http://xxxx-service/xxxx?ids={ids}",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<String>() {},
                CollUtils.join(ids, ","));
    }
}
```

### 方法三
使用DiscoveryClient工具实现服务发现,SpringCloud已经帮我们自动装配,我们可以直接注入使用
```java
@Service
public class RemoteCallService {
    @Autowired
    private DiscoveryClient discoveryClient;

    public String call() {
        // xxxx-service是提供者的服务名
        List<ServiceInstance> instances = discoveryClient.getInstances("xxxx-service");
        if (CollUtils.isEmpty(instances)) {
            return null;
        }
        ServiceInstance instance = instances.get(0);
        return restTemplate.exchange("http://" + instance.getHost() + ":" + instance.getPort() + "/xxxx?ids={ids}",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<String>() {},
                CollUtils.join(ids, ","));
    }
}
```