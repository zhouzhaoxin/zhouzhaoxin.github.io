---
layout: post
title: "Spring Boot踩坑之路三"
author: zhouzhaoxin
categories: ["后端"]
image: assets/images/8.jpg
comments: false
---
使用JPA进行CRUD以及IDEA 中 Devtools热部署配置

## Spring Boot进行简单的CRUD
只需要定义一个repository继承PagingAndSortingRepository、PagingAndSortingRepository或JpaRepository就可以使用Spring Data为我们提供的CRUD。三个接口都是做数据操作的，下面解释了他们三个的区别，我们应该根据需求灵活使用。

JpaRepository继承了PagingAndSortingRepository，PagingAndSortingRepository继承了CrudRepository

他们的主要功能是：

CrudRepository 提供主要的CRUD方法

PagingAndSortingRepository 提供分页和排序的方法

JpaRepository 提供一些JPA关联方法，比如批量删除数据

因为上述的继承关系，JpaRepository拥有PagingAndSortingRepository和CrudRepository 的所有方法。所以如果你不需要JpaRepository和PagingAndSortingRepository提供的方法的话，请使用CrudRepository

## Spring Boot IDEA 热部署
首先要对IDEA进行必要配置，否则热部署不起作用

## IDEA自动构建
#### 修改IDEA注册信息
输入命令Ctrl + Shift + A 然后搜索registry回车

#### 添加devtools依赖
添加依赖，重启项目，然后试着修改java文件，自动重新部署成功
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```