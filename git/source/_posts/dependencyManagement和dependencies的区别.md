---
title: dependencyManagement和dependencies的区别
date: 2025-07-22 08:22:06
tags:
- Java
- Web
- SpringBoot
categories:
- JavaWeb
comments: true
---
dependencyManagement和dependencies都是依赖相关的标签,但是不能互相替代

<!-- more -->

# dependencies
dependencies是用来声明项目的依赖,在项目中使用到的依赖都需要在dependencies中声明
```xml
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
        <version>3.17.4</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

# dependencyManagement
dependencyManagement是用来管理项目的依赖版本,在项目中使用到的依赖都需要在dependencyManagement中声明,一般写在父项目.
```xml
<properties>
    <spring-boot.version>2.6.13</spring-boot.version>
    <jjwt.version>0.9.1</jjwt.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```