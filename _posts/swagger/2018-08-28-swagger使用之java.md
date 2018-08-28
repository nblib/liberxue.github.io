---
layout: blog
istop: false
title: "swagger使用之java"
description: "swagger使用"
date:  2018-08-28
category: swagger
tags:
- 自动文档
- swagger
---

# swagger使用之java

## swagger介绍

简单来说,swagger用于自动生成接口文档,文档自动更新,在线测试接口.

### 手写接口的缺点
* 写好接口项目后,需要手写文档,然后发送给调用方查看
* 当手写接口时,需要手动指明请求参数,返回参数,参数的说明,接口的调用地址等等,这些如果在接口注释写明的话,还需要复制出来
* 当接口变动时,手写文档需要同步更改
* 不能在线测试,文档写的再好,也需要测试一下才能更加领会,通常测试接口需要第三方工具,比如postman
* 当接口数量多的情况下.....

### swagger的优点
* 在接口上按照swagger标准书写说明,既写了方法注释,又可以生成文档
* 请求返回参数只要注释写到好,文档样样看的清
* 内置测试工具,看了文档还能在线测试
* 接口有变动,自动更新文档
* 使用swagger,所有人的文档格式统一,美观

## swagger简单使用

### 环境
* springboot
* java

### 开始使用

#### 环境配置
首先新建一个springboot项目
```
demo-swagger-java
```

##### 添加依赖包

新建好项目后,在pom中添加依赖:
```
 <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.8.2</version>
</dependency>
</dependencies>
```
* 使用springfox的swagger2版本
* swagger-bootstrap-ui 相比于原生的页面好看,功能多一点

##### 配置springboot的配置文件

然后需要配置spring的配置文件.spring的配置文件建议修改为`.yml`格式,这样方便配置和查看.一个项目建议最好包含三个springboot的配置文件:
* `application.yml`: 这个文件为新建springboot项目时自带的配置文件(有可能后缀为`.properties`,需要自己改为`.yml`).springboot加载配置时首先寻找这个文件
* `application-dev.yml`: 这个配置文件为开发模式下使用.swagger一般也就开发和测试时使用,上线时如果使用的话会对外暴露.有风险
* `application-prod.yml`: 线上模式使用这个配置文件.这个文件中不要包含swagger的配置
* `application-test.yml`: 这个文件可选的存在,一般有测试环境时可以使用这个配置文件

> 为什么用分这么多配置文件
>> 一个项目包含开发,测试,上线等状态,日志输出位置,数据库连接等配置在不同状态下都可能不同,如果只放到一个配置文件中,需要来回修改

使用多个配置文件时,想要改变项目启动时使用哪个配置文件,通过在`application.yml`中修改:
```
spring:
  profiles:
    active: dev
```
* `dev`为上面提到的配置文件的后缀,比如`application-dev.yml`即为`dev`,对应的其他的为`prod`,`test`
* 通过设置这个值,可以改变项目启动时加载哪个配置文件

##### 使用`demo-swagger-java`的配置

由于目前没有发现springboot自动配置swagger的项目,自己封装了一个swagger基于配置文件的简单配置.参考 [demo-swagger-java](https://github.com/nblib/demo-swagger-java)  
其中:
* config/swagger: swagger的配置文件.里面详细注释了各个步骤的作用
* config/swaggerproperties: 自定义的配置属性,里面的字段都可以在springboot的配置文件中配置.

使用时可以把这两个文件单独复制下来放到自己的项目中使用.**使用前务必参考README文件说明**

###### 使用方法
当需要在项目中使用`demo-swagger-java`中的配置方式时,需要:
* 复制`config`文件夹下的所有到自己项目中,放到哪里需要自己定义,建议所有的配置类统一放到一个文件夹下
* 封装的配置在`dev`和`test`模式下才会激活,也就是,swagger的配置必须放到`application-dev.yml`或`application-test.yml`配置文件中,并且在`application.yml`中配置`spring.active`为`dev`或`test`模式

配置说明,在`application-dev.yml`或`application-test.yml`中添加:
```
swagger:
  basepackage: demo.nblib.swagger.controller #接口所在包,用于扫描,供swagger提供文档
  version: 1.0  # 接口版本
  serviceUrl: test.nblib.org  # 接口的地址
  globalParms:    # 当需要用到全局参数时,也就是每个接口都必须的参数时,指定这个配置项
    - name: _appid  # 参数的名称
      desc: 来源的app类型  # 参数描述
      type: String  #参数类型,比如String,int,boolean等等
      target: query # 参数被放到哪个位置:比如请求时加入到header中,或者作为请求参数: 可选值: query,header
      def: pc  # 默认值
      required: true # 是否必须
```
* globalParms 为可选配置,主要用来定义一些参数供所有接口使用.比如在拦截器中配置了需要_appid这个参数,也就是调用所有接口必须有这个参数,如果在所有接口中说明,太麻烦.这中情况下,可以在这里定义

通过以上配置,swagger基本可以使用.可以启动命令通过访问配置文件定义的端口来访问: https://localhost:8080/doc.html ,

#### 使用案例
通过以上配置,使用swagger的基本条件已经具备,下面列出一些场景.

##### 


