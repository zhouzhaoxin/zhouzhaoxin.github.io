---
layout: post
title: "Spring Boot踩坑之路一"
author: zhouzhaoxin
categories: ["后端"]
image: assets/images/11.jpg
comments: false
---
> Takes an opinionated view of building production-ready Spring applications. Spring Boot favors convention over configuration and is designed to get you up and running as quickly as possible.

旨在简化创建产品级的 Spring应用和服务。Spring Boot 引导优先于配置，它可以让你避免繁杂的配置，尽可能的帮助你快速建站。

## 为什么使用Spring Boot
经过十多年的发展Spring家族已经壮大，要灵活使用Spring家族的产品已经变得有些困难，尤其是要维护一大堆的配置文件，在项目开发中令人头疼。Spring Boot解决了这个问题，并大大简化了我们的开发成本

## 其优点如下:
- 不用看一大坨的xml。用java config可以让你很容易明白一些框架的关键
- Spring Boot 要解决的问题, 精简配置是一方面, 另外一方面是如何方便的让spring生态圈和其他工具链整合(比如redis, email, elecsearch)
- 配合各种starter使用，基本上可以做到自动化配置
- 配合Maven或Gradle等构件工具打成Jar包后，Java -jar 简化部署运行

## 建立maven web项目
使用maven建立web项目，并参考官方文档进行版本选择和pom配置

官方maven配置，请根据需求选择版本

POM中添加parent标签
添加parent后添加相关依赖不需要version

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.2.RELEASE</version>
</parent>
```
## 添加依赖
web工程的依赖，包括spring mvc tomcat等，spring boot会在需要时使用

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
## 添加插件
用来在main方法中启动工程

```xml
<plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin </artifactId>
    </plugin>
</plugins>
```
## 编写代码 HELLO WORLD
```java
// 其中@SpringBootApplication申明让spring boot自动给程序进行必要的配置，等价于使用@Configuration，@EnableAutoConfiguration和@ComponentScan
@RestController
@SpringBootApplication
public class HelloWorld {
    @RequestMapping("/")
    public String hello() {
        return "Hello World";
    }
    public static void main(String[] args) {
        SpringApplication.run(HelloWorld.class,args);
    }
}
```
检验成果
运行main函数之后访问http://localhost:8080/即可看到结果

## 我遇到的问题
问题出现在我建立maven项目编写java代码的时候在默认包中写的application类，并没有建立包。这种做法让Spring Boot每次都会扫描默认类及下属的所有类，浪费大量时间。所以在启动时会报警告，启动不成功

Your Application class should be placed in a specific package and not in the default (top-level) package. For example, put it in com.example and place all your application code in this package or in sub-packages like com.example.foo and com.example.bar.

Placing your Application class in the default package, i.e. directly in src/main/java isn’t a good idea and it will almost certainly cause your application to fail to start. If you do so, you should see this warning:
```text
** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.
```