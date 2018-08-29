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
    <exclusions>
        <exclusion>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-annotations</artifactId>
        </exclusion>
        <exclusion>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.5.21</version>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
    <version>1.5.21</version>
</dependency>
```
* 由于这个版本使用swagger1.5.20,这个版本有个bug就是int类型参数回报警告异常,所以排除掉,单独引入
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
**请参照:[demo-swagger-java](https://github.com/nblib/demo-swagger-java)**,建议下载并运行起来,可以查看运行效果


通过以上配置,使用swagger的基本条件已经具备,下面列出一些场景.首先介绍新建一个controller的类:
```
@Api(value = "不同参数不同返回值得使用方法,这里举例了不同参数,不同返回值的类型的使用方法", tags = "参数测试")
@RestController
@RequestMapping("/parm")
public class ParmTestController {
```
* Api: 说明这个类也就是controller的介绍,value用于描述这个类的用途,tags用于分类,相当于命名

##### 无参数字符串返回值
```
@ApiOperation(value = "无参string", notes = "无参数返回String类型")
@ApiResponses({@ApiResponse(code = 200, message = "请求成功,返回字符串", response = String.class), @ApiResponse(code = 404, message = "请求地址或方法不对")})
@GetMapping("/noparm")
public String noParm() {
    return "nihao";
}
```
* ApiOperation用于对方法的说明.
    * value: 方法命名.也就是为方法起个名.简短
    * notes: 方法描述.描述方法的用途等等
* ApiResponses用于描述响应状态的含义,里面存放多个响应描述
    * ApiResponse用于描述一个响应状态的含义,比如响应状态码200(code)的含义是什么(message),响应的数据类型是什么(response)

这个例子中,是一个get方法,没有参数.最后返回一个字符串.

##### 有参数有返回值
```
@ApiOperation(value = "有参string", notes = "有参数返回String类型")
@ApiResponses({@ApiResponse(code = 200, message = "请求成功,返回字符串", response = String.class), @ApiResponse(code = 404, message = "请求地址或方法不对")})
@GetMapping("/parmm")
public String parm(@ApiParam(name = "name", example = "nblib") @RequestParam("name") String name) {
    return "nihao: " + name;
}
```
* ApiParam用于参数说明
    * name为说明参数的名称,一般不用指定,默认会自动获取参数名
    * example: 举个例子,比如上面的nblib为一个name参数的例子

如果不想在每个参数前面添加ApiParm,可以通过`ApiImplicitParams`来指定.比如下面的例子

##### 请求包含请求头参数
```
@ApiOperation(value = "请求头说明", notes = "参数包含请求头")
@ApiImplicitParams({
        @ApiImplicitParam(name = "token", value = "请求头中包含的参数", paramType = "header"),
        @ApiImplicitParam(name = "age", value = "参数age", paramType = "query")
})
@GetMapping("/header")
public ReqEntity<String> header(@RequestHeader("token") String token, @RequestParam("age") Integer age) {
    ReqEntity<String> req = new ReqEntity<>();
    req.setCode(1);
    req.setMsg("success");
    req.setData("post is :" + token + age);
    return req;
}
```
* ApiImplicitParams用于同一参数描述,存放多个ApiImplicitParam
* ApiImplicitParam描述一个参数的含义
    * name: 参数的名称,一般不用指定
    * value: 参数的描述信息
    * paramType: 参数的类型,是请求头参数,还是请求参数,还是表单参数

##### 自定义返回值类型和范型
```
@ApiOperation(value = "范型返回结果", notes = "测试使用范型的数据类型作为返回值")
@GetMapping("/pattern")
public ReqEntity<CarInfo> pattern() {
    ReqEntity<CarInfo> req = new ReqEntity<>();

    CarInfo carInfo = new CarInfo();
    carInfo.setInfoId("234");
    carInfo.setPrice(345.22f);
    carInfo.setSpecName("宝马");
    req.setData(carInfo);
    req.setMsg("success");
    req.setCode(1);
    return req;
}
```
ReqEntity的定义:
```
/**
 * 响应结果集
 */
public class ReqEntity<T> {

    @ApiModelProperty(value = "响应信息")
    private String msg;

    @ApiModelProperty(value = "响应状态码", allowableValues = "0,1,-1")
    private Integer code;

    @ApiModelProperty(value = "返回对象")
    private T data;

    //get和set方法省略.....
}

```
CarInfo的定义:
```
public class CarInfo {
    @ApiModelProperty("汽车的id")
    private String infoId;
    @ApiModelProperty("汽车的品牌")
    private String specName;

    @ApiModelProperty(value = "汽车价格", allowableValues = "range[0,9999999]")
    private Float price;
}
```
* ApiModelProperty用于对自定义类型的字段说明,这样当作为返回值时,可以解析到文档中,从而可以看到每个返回值得含义


**更多例子,请参照:[demo-swagger-java](https://github.com/nblib/demo-swagger-java)**,建议下载并运行起来,可以查看运行效果

## 常用注解说明

* @Api()用于类: 生成这个类的说明文档
* @ApiOperation()用于方法: 生成对这个方法的说明文档
* @ApiParam()用于方法，参数，字段说明: 生成对参数的说明文档
* @ApiModel()用于类: 自定义实体的说明文档
* @ApiModelProperty()用于方法，字段: 自定义实体中的字段的说明文档
* @ApiIgnore()用于类，方法，方法参数: 忽略这个接口的说明文档
* @ApiImplicitParam() 用于方法: 单独对一个参数进行说明
* @ApiImplicitParams() 用于方法: 包含多个ApiImplicitParam
* @ApiResponse 用于方法: 对返回状态进行说明
* @ApiResponses 用于方法: 包含多个返回状态说明

下面详细介绍

### @Api()
用于类,对这个类进行生成文档,常用配置有
* value: 对类的描述,描述这个类的用途等等
* tags: 对这个类进行分组,这样显示的时候相同tag会被放到一个目录中,方便查看
### @ApiOperation()
用于接口方法,定义这个接口的描述名称等信息,常用配置:
* value: 接口的简短描述
* notes: 接口的详细描述

### @ApiParam()
生成对参数的说明文档.常用:
* name: 参数的名称,一般不用设置,会自动根据接口定义生成
* value: 对参数的简介
* required: 说明这个参数是否是必须的
* hidden: 是否在文档中隐藏这个参数的说明
* example: 为参数举个例子,更容易理解这个参数

### @ApiModel
有时候接口返回结果为自定义的数据类型,这个注释用于对自定义类型的字段等进行描述,
* value: 自定义类型的名称,默认使用类名
* description: 详细描述自定义类型的用途

### @ApiModelProperty
对自定义类型中的字段进行描述,这样生成文档会显示每个字段的含义
* value: 对字段简单的描述
* require: 这个字段是否是必须的

### @ApiImplicitParam
用于说明一个参数的含义.
* value: 描述这个参数的含义
* required: 这个参数是否是必须的,默认为false
* paramType: 这个参数的类型,是在url中传的,还是作为参数,或表单,或在请求头中,可用值为: path,query,body,header,form
* @ApiImplicitParams
统一管理多个参数,值为多个ApiImplicitParam,比如
```
 @ApiImplicitParams({
            @ApiImplicitParam(name = "token", value = "请求头中包含的参数", paramType = "header"),
            @ApiImplicitParam(name = "age", value = "参数age", paramType = "query")
    })
```

### @ApiResponse
对响应状态的含义描述,比如描述一个200状态码的含义等等
* code: 响应状态码
* message: 这个状态码的含义
* response: 响应这个状态的返回值类型比如:String.class

### @ApiResponses
统一管理多个响应状态,比如
```
@ApiResponses({@ApiResponse(code = 200, message = "请求成功,返回字符串", response = String.class), @ApiResponse(code = 404, message = "请求地址或方法不对")})
```
