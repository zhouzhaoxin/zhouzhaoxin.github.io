---
layout: post
title: "Spring Boot踩坑之路二"
author: zhouzhaoxin
categories: ["后端"]
image: assets/images/10.jpg
comments: false
---
上篇文章使用Maven构建项目并成功运行hello，让我初步感受Spring Boot的强大。本次我将使用更优雅的方式创建Spring Boot项目并学习它对数据库的相关操作

## 更优雅的创建Spring Boot项目
上篇文章的方法是建立Maven项目、配置pom.xml、建立application类。当用到的开源项目多的时候，每次配置pom.xml很会很麻烦。使用IDEA为我们提供的Spring Initializr可以快速创建Spring Boot项目，我们将不再需要之前繁琐的步骤

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.app</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
## Spring Boot JPA
回归主题，Spring Boot是如何操作数据库的呢，本次以Oracle数据库为例介绍Spring Boot的JPA操作（Hibernate版）

## 连接数据库
添加依赖(需要添加两个依赖，一个是Spring Boot操作数据库的依赖，还有一个是数据库驱动)

 ```xml
 <!--Spring Boot java persist API dependency-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <!-- Oracle数据库驱动 -->
  <!-- Oracle Driver Dependency which located in local repository -->
  <dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0</version>
</dependency>
<!-- MySql 驱动，maven官方就有，无需本地添加
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
</dependency>
-->
```
## 注意
由于商业版权和版本的问题，Maven中心仓库并不支持Oracle驱动包，所以需要根据自己的Oracle进行手动安装

首先从Oracle安装目录中找到本地Oracle驱动，以我为例：

E:\app\Administrator\product\11.2.0\dbhome_1\jdbc\lib
进入我的路径将ojdbc添加到本地Maven仓库

mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=ojdbc6.jar
-- 指定坐标、打包格式、打包文件
回车安装

## 添加属性配置
加载了SpringBoot连接和驱动依赖当然不能让我们连接到数据库，我们还需要告诉Spring Boot一些数据库连接信息

我们只需要在application.properties文件中添加就可以了

## 基本的连接信息，指定数据源
spring.datasource.url = jdbc:oracle:thin:@127.0.0.1:1521:orcl
spring.datasource.username = scott
spring.datasource.password = orcl
spring.datasource.driverClassName = oracle.jdbc.OracleDriver
## java持久化API Hibernate配置模式
spring.jpa.hibernate.ddl-auto = update
新建实体类，并交给Hibernate 维护
```java
package com.example.entity;
import javax.persistence.*;
@Entity
@Table(name = "demo_user")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE,generator = "seq_user")
    @SequenceGenerator(name = "seq_user",sequenceName = "seq_user")
    private int id;
    private String userName;
    private String password;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```
最后启动项目，是不是发现数据库里面多了个demo_user表了呢

## 遇到的问题
安装完成ojdbc驱动在pom.xml中引用仍然无效

```text
创建项目的时候选择了默认C盘.m2下的仓库，而我的Maven仓库配置在D盘，所以安装的驱动的时候安装到了D盘的仓库，使用的却是系统默认的仓库，尴尬-_-!
```

在全部配置好之后启动工程会把表建立出来，但是当再次启动的时候竟然报了对象已由现有名称引用异常

```text

通过log观察，他又执行了一次建表语句。我多次检查自己的配置，确实是spring.jpa.hibernate.ddl-auto = update没问题。到网上查了半天，也没找到有相似问题的结果。出去吹个风，回来发现我需要引用的驱动jar包是ojdbc6，而在pom中引入的是服务器数据库的ojdbc14。白白浪费了这么长时间<(￣3￣)> 。
```