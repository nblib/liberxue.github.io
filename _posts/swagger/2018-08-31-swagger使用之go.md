---
layout: blog
istop: false
title: "swagger使用之go"
description: "swagger使用"
date:  2018-08-29 16:38
category: swagger
tags:
- 自动文档
- swagger
---


# swagger使用之go

swagger介绍参考: [swagger使用之java](https://www.nblib.org/2018/08/29/swagger%E4%BD%BF%E7%94%A8%E4%B9%8Bjava.html)

## 环境
* golang1.9,1.10
* beego

## 开始使用
使用步骤如下:
* 新建beego项目
* 编辑router.go
* controller方法中加入注释
* 在main.go中加入配置
* 使用bee命令启动
### 项目搭建
新建一个beego项目
```
demo_swagger
```
### 配置swagger
使用swagger,router配置,必须使用如下格式:
```
func init() {
	ns := beego.NewNamespace("/v1",
		beego.NSNamespace("/demo",
			beego.NSInclude(
				&controllers.MainController{},
			),
		),
	)
	beego.AddNamespace(ns);
}
```
* 所有接口放到Namespace中
* 一个Namespace下可以有多个namespace
* 所有controller必须使用NSInclude导入

编写`router/router.go`文件:
```
// @APIVersion 1.0.0
// @Title 测试使用swagger
// @Description 测试使用swagger的演示样例
// @Contact tingshage@163.com
package routers

import (
	"demo_swagger/controllers"
	"github.com/astaxie/beego"
)

func init() {
	ns := beego.NewNamespace("/v1",
		beego.NSNamespace("/demo",
			beego.NSInclude(
				&controllers.MainController{},
			),
		),
	)
	beego.AddNamespace(ns);
}
```
* 必须在文件最开头放置注释.这个注释是对整个项目的介绍.
    * APIVersion: 当前项目的接口版本
    * Title: 当前项目的名称
    * Description: 项目的简单描述
    * Contact: 作者,可以不写
* 路由必须使用这样的格式书写

配置好路由后,也添加了对项目的描述文档后,现在开始配置一个controller:
```
//编辑controller/default.go
type MainController struct {
	beego.Controller
}

// @Title demoGet
// @Description 测试get方法
// @Param   name     query    string  true        "测试name"
// @Param   addr     query    string  true        "测试addr"
// @Success 200 sucees
// @Failure 400 为上传参数或错误参数
// @Failure 404 not found
// @router /info [get]
func (c *MainController) Get() {
	name := c.GetString("name")
    addr := c.GetString("addr")
    if strings.EqualFold(name, "") {
        c.Abort("400")
    }
    if strings.EqualFold(addr, "") {
        c.Abort("400")
    }
    c.Data["json"] = "test"
    c.ServeJSON()
}

// @Title getAge
// @Description 测试自定义get方法
// @Success 200 sucees
// @Failure 404 not found
// @router /age [get]
func (c *MainController) GetAge()  {
	c.Data["json"] = "13"
	c.ServeJSON()
}

```
* 一个方法使用@router注解形式配置router
* 结合router中配置的路由,如上Get方法的路由最终为`http://localhost:8080/v1/demo/info`,GetAge方法的路由为:`http://localhost:8080/v1/demo/age`
* 文档注解的含义:
    - Title: 这个接口方法的名称
    - Description: 描述
    - Param: 表示需要传递到服务器端的参数，有五列参数，使用空格或者 tab 分割，五个分别表示的含义如下:
        - 第一列: 参数的名称
        - 第二列: 参数的类型.可以有的值是 formData、query、path、body、header，formData 表示是 post 请求的数据，query 表示带在 url 之后的参数，path 表示请求路径上得参数，例如上面例子里面的 key，body 表示是一个 raw 数据请求，header 表示带在 header 信息中得参数。
        - 第三列: 参数类型.string,int等等
        - 第四列: 是否必须
        - 第五列: 参数描述
    - Success: 成功返回给客户端的信息，三个参数，第一个是 status code。第二个参数是返回的类型，必须使用 {} 包含，第三个是返回的对象或者字符串信息，如果是 {object} 类型，那么 bee 工具在生成 docs 的时候会扫描对应的对象，这里填写的是想对你项目的目录名和对象，例如 models.ZDTProduct.ProductList 就表示 /models/ZDTProduct 目录下的 ProductList 对象。
    - Failure: 失败返回的信息，包含两个参数，使用空格分隔，第一个表示 status code，第二个表示错误信息
    - router: 路由信息，包含两个参数，使用空格分隔，第一个是请求的路由地址，支持正则和自定义路由，和之前的路由规则一样，第二个参数是支持的请求方法,放在 [] 之中，如果有多个方法，那么使用 , 分隔。

如上,是简单的需要生成文档的注释,**使用`router`不仅用于生成文档主要还是注解形式的router设置.可以看[注解路由](https://beego.me/docs/mvc/controller/router.md)**,

这里只是简单列出一些,具体可以参考官方的说明,官方文档比较详细.以上部分内容copy自[官方文档](https://beego.me/docs/advantage/docs.md)

主要问题在于启动应用和查看文档.这里主要说明

### 启动

首先说明的是,bee启动自动化文档时都做了什么:
* 查找编译带有规定格式的注释的方法,在`router`文件夹下生成`commentsRouter_controllers.go`文件
* 将所有文档编译生成为`json`和`yml`格式,放到项目的根目录`swagger`文件夹下,如果没有这个文件夹会自动创建
* 从GitHub上下载图形化界面,并自动解压,将网页放到`swagger`文件夹下
* 启动应用,访问指定路径,查看文档

如上所述.beego将所有生成的文档和查看文档的页面放到了`swagger`目录下,但是,现在我们通过浏览器是访问不到这个目录的.所以要先配置这个路径:
```
//编辑main.go

func main() {


	if beego.BConfig.RunMode == "dev" {
		beego.BConfig.WebConfig.DirectoryIndex = true
		beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
	}

	beego.Run()
}

```
* 首先判断配置文件中的`runmode = dev`,也就是`app.conf`中的`runmode`,是否为dev,这个是个人添加判断,可以不加判断,但是一般文档用于测试时查看
线上环境不需要查看文档.所以这里添加了判断.
* 通过设置将浏览器访问路径`/swagger`映射到swagger这个文件夹,也就是将swagger变为一个静态目录,里面的内容对外开放.

配置好后,接下来可以启动应用:
```
//在控制台,使用bee启动命令
bee run -gendoc=true -downdoc=true
```
* bee是beego使用的工具,如果没有这个命令:参考这个文档: [bee工具使用](https://beego.me/docs/install/bee.md)
* gendoc: 自动生成文档
* downdoc: 如果本地没有图形化网页,自动下载并解压到swagger文件夹

如上,启动后,可以看到控制台输出:
```
localhost:demo_swagger hewe$ bee run -gendoc=true -downdoc=true
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.9.1
2018/08/31 10:14:33 INFO     ▶ 0001 Using 'demo_swagger' as 'appname'
2018/08/31 10:14:33 INFO     ▶ 0002 Initializing watcher...
demo_swagger/routers
demo_swagger
2018/08/31 10:14:34 INFO     ▶ 0003 Generating the docs...
2018/08/31 10:14:34 SUCCESS  ▶ 0004 Docs generated!
2018/08/31 10:14:35 SUCCESS  ▶ 0005 Built Successfully!
2018/08/31 10:14:35 INFO     ▶ 0006 Restarting 'demo_swagger'...
2018/08/31 10:14:35 SUCCESS  ▶ 0007 './demo_swagger' is running...

```
通过访问:`http://lcoalhost:8080/swagger`查看文档页面,这个路径对应于`main`方法中的`StaticDir`配置路径.

### 问题
查看文档网页时.页面右下角可能有一个红色的提示:`ERROR {...}`,点击后会看到:
```
{"schemaValidationMessages":[{"level":"error","message":"Can't read from file swagger.json"}]}
```
这种情况是因为,生成的文档会调用swagger官方检查生成的文档文件格式是否正确.这里可以忽略这个问题.


### 例子
可以查看例子: [demo_swagger](https://github.com/nblib/demo_swagger)