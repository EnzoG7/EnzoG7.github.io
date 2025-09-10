---
title: JavaWeb的Bug之SpringBoot
date: 2025-08-17 10:41:29
tags: 
- SpringBoot
- Java
- Web
- Bug
categories:
- Bug
comments: true
---
本文记录SpringBoot的一些Bug和部分解决措施.

<!-- more -->

# 未知异常 报错在Controller某个接口上
## 问题描述
在使用SpringBoot开发时,某个Controller接口抛出未知异常,报错信息抛在接口地址上,但问题实际在Service层.
业务逻辑是上传pdf到OSS,然后再把OSS的地址填到DTO去调用大模型评测.

## 问题排查
表面错误是pdf上传失败,第二次上传成功,然后到后端调用大模型评测时,报未知异常.
一步步排查发现调用RedisTemplate时,错误在未指定泛型<String, String>.
该错误与报错信息没有任何关联,很难排查.

## 解决方法
指定RedisTemplate的泛型为<String, String>,问题解决.