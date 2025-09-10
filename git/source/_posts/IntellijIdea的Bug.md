---
title: Intellij Idea的Bug
date: 2025-07-14 08:44:13
tags:
- Java
- Bug
categories:
- Bug
comments: true
---
本文记录Intellij Idea的一些Bug和部分解决措施.

<!-- more -->
# Directory '.../ProjectNamePath/...' does not contain a Gradle build.
## 问题出现的部分原因
1.在创建product模块时,作为gradle项目创建,后来改成maven项目,但IDEA认为该模块是Gradle项目,在尝试编译时寻找Gradle配置文件.
2.之前创建的项目直接将文件夹删除了,并没有清除记录
## 解决方法
找到对应项目目录下的gradle.xml文件,一般在'.../ProjectNamePath/.idea/gradle.xml'.将其中涉及项目的<option name = "...">都注释掉.