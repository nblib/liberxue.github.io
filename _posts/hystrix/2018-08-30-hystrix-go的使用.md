---
layout: blog
istop: false
title: "hystrix-go的使用"
description: "hystrix-go,go hystrix"
date:  2018-08-30
category: hystrix
tags:
- hystrix
- go
---

# hystrix-go 的使用

## hystrix
### hystrix简介
在分布式系统中,调用一个接口,可能这个接口会调用多个接口,如果被调用的多个接口中,有个别接口出现响应慢,或者运行异常多次返回错误,
此时会导致调用方的执行时间变长,当有大量请求进入时,整个系统的压力会变大,导致其他接口不能正常使用.
### hystrix功能
* 超时机制.当被调用接口在指定时间内没有返回,那么hystrix将自动终止执行.返回超时错误.避免调用方过长时间等待
* 错误检测机制.当被调用方返回错误的次数变多,超过一定阈值.hystrix会跳闸.此时不会再调用发生错误的接口.

### hystrix的breaker

![hystrix_breaker](https://nblib.github.io/style/images/hystrix_struct.png)

如上图,hystrix执行真正逻辑时,使用的事breaker进行执行.也就是调用Do()方法传入的name即为breaker的命名,每个命名对应
一个breaker
* hystrix下可以有多个breaker
* 每个breaker独享一个配置文件
* 每个breaker用来执行真正的逻辑

建议使用时,每个接口提供方使用各自的breaker,这样才能真正实现远程接口不可用时的对应断路.比如:
```
hystrix.Do("api1",run,fallback)
hystrix.Do("api2",run,fallback)
hystrix.Do("api2",run,fallback)
```
同时也可以针对每个breaker单独设置配置项.

## hystrix-go

go版本的hystrix,一下的样例程序: [demo_hystrix](https://github.com/nblib/demo_hystrix)

### 环境准备
* go1.10
* hystrix-go

新建一个web项目,我这里使用的是beego:
```
demo-hystrix
```
导入`hystrix-go`的包:
```
	"github.com/afex/hystrix-go/hystrix"

```
为模拟远程接口的超时和出错.这里新建一个controller.可以通过参数控制超时和出错:
```
type DemoController struct {
	beego.Controller
}

//测试接口,通过参数控制执行时间和返回结果.
// 比如: to=1,err=0,那么这个接口会执行十秒然后返回. to=0,err=1,那么这个接口会返回一个401状态码
//参数:
// to: 是否开启time.Sleep.也就是让线程等待10秒.值为: 1开启,0不开启
// err: 是否返回错误.1返回,0不返回.
func (c *DemoController) Get() {
	isTo, err := c.GetBool("to")
	if err != nil {
		c.Ctx.WriteString("错误参数 to")
	}
	isErr, err := c.GetBool("err")
	if err != nil {
		c.Ctx.WriteString("错误参数 err")
	}

	if isTo {
		time.Sleep(10 * time.Second)
	}
	if isErr {
		c.Abort("401")
	}
	c.Ctx.WriteString("你好")
}

```
配置router.go,配置路由为`/demo`
然后设置配置,使用默认的配置值,在main.go中添加:
```
var config = hystrix.CommandConfig{

    Timeout: 1000, //超时时间,调用接口最多等待时间,超时了,立马返回,不等待接口  单位毫秒

    MaxConcurrentRequests: 10, //最大请求数.同时调用这个接口的最大线程数

    SleepWindow: 5000, //当断路后,等待多长时间，熔断器会再次放行一个调用线程去验证是否恢复。单位毫秒

    ErrorPercentThreshold: 50, //错误率,当远程接口返回错误的数量/总调用次数,大于这个百分比,会断路.比如调用100次,有十次以上是错误

    RequestVolumeThreshold: 20, //请求阈值  熔断器是否打开首先要满足这个条件；这里的设置表示至少有5个请求才进行ErrorPercentThreshold错误百分比计算
}
hystrix.ConfigureCommand("test", config)
```
### 开始使用
在上面步骤配置好后,开始简单的使用.
#### 使用介绍
```cgo
func Do(name string, run runFunc, fallback fallbackFunc) error
func Go(name string, run runFunc, fallback fallbackFunc) error
```
当需要调用远程接口时,基本调用这两个方法足以.两者的区别是Do方法是同步的,Go方法是异步的.一般同步的就可以满足需求.参数说明:
* name: 为调用起个名字,一般每个远程接口使用一个名字,相同接口使用相同的名字,方便管理.而且相同名字共享同一个config,也就是上面的配置
* run: 真正执行的方法.如果调用远程接口,则在这个方法中调用.这个方法没有参数,返回一个errror
* fallback: 当出现错误时,hystrix会调用这个方法,这个方法参数为一个error,返回值为一个error

run方法示例:
```
func run() error{

    return nil
}
```
fallback方法示例:
```
func fall(err error) error{
    //...
    return <nil>
}
```
#### 调用接口例子
这里写个例子,调用上面建的那个模拟接口:
```
type MainController struct {
	beego.Controller
}
// 测试 hystrix,参数用来控制远程接口的行为,来测试hystrix
// 比如: to=1,err=0,那么远程接口会执行十秒然后返回. to=0,err=1,那么远程接口会返回一个401状态码
//参数:
// to: 是否开启time.Sleep.也就是让线程等待10秒.值为: 1开启,0不开启
// err: 是否返回错误.1返回,0不返回.
func (c *MainController) Get() {
	//获取参数
	isTo := c.GetString("to")
	isErr := c.GetString("err")

	// 一般情况不要调用,这个方法执行完后,会清理hystrix的状态,和断路器的状态
	//defer hystrix.Flush()

	//breaker.IsOpen()和breaker.AllowReques()不要调用,会导致断路器状态改变.调用可以在执行完成后调用
	//打印: isOpen: 断路器是否打开,刚开始是false,表示没有断路,可以正常访问.AllowRequest,是否允许请求
	//fmt.Println(breaker.IsOpen(), breaker.AllowRequest())

	//Do方法会在当前线程内执行
	err := hystrix.Do("test", func() error {
	    //打印判断是否执行这个方法
	    fmt.Println("start exec.........")
		resp, err := http.Get("http://localhost:8080/demo?" + "to=" + isTo + "&" + "err=" + isErr)
		if err != nil {
			return err
		}
		defer resp.Body.Close()
		if resp.StatusCode == 401 {
			return errors.New("err occur 401")
		}
		bytes, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			return err
		}
		c.Ctx.WriteString(string(bytes))
		//return errors.New("not allow")
		return nil
	}, func(e error) error {
		if e != nil {
			fmt.Println("error: ", e.Error())
		}
		return e
	})
	
	//可以在执行完后调用
	breaker, _, _ := hystrix.GetCircuit("test")
	fmt.Println("OPEN: ",breaker.IsOpen(),"ALLOW: ",breaker.AllowRequest())
	if err != nil {
		c.Abort("401")
	}
}

# 修改router.go,配置路由
/index
```
* isTo和isErr: 浏览器传过来的参数,用于控制hystrix中调用的远程的接口行为,模拟超时和错误返回
* hystrix.Do: 
    * test: 为这个调用起个名字,最好相同接口具有相同的名字
    * runFunc: 执行逻辑调用的方法
    * fallbackFunc: 发生错误时的回调方法.可以在这里记录日志等等.
* hystrix.GetCircuit: 获取这次执行的breaker,测试时使用,查看断路器的状态


这里的执行逻辑:
* 首先获取参数,用来控制远程接口模拟超时或出错
* 将调用远程接口的逻辑封装到hystrix.Do方法中.
* 远程调用逻辑为: 使用httpclient调用远程接口,如果返回错误,就直接返回,如果返回状态码为401,也返回一个错误,这里是为了模拟
,接下来读取返回结果,复制给当前接口的返回值.最后没有错误的情况下返回nil
* 当出现错误时,在fallback中打印一下,然后将错误返回.
* 调用hystrix.Do后,最后返回一个err.就是hystrix.Do的fallback返回的错误.
* 通过获取Circuit获取breaker.通过IsOpen获取当前是否断路,true为断路.通过AllowRequest判断是否可以调用.当断路时,会在SleepWindow时间后尝试调用一下

#### 测试
##### 正常情况

首先看下正常调用情况,通过浏览器访问: `http://localhost:8080/index?to=0&err=0`,查看结果:
```
# 控制台输出
start exec.........
2018/08/30 14:20:43.708 [D] [server.go:2694]  |      127.0.0.1| 200 |     43.583µs|   match| GET      /demo   r:/demo
OPEN:  false ALLOW:  true
2018/08/30 14:20:43.708 [D] [server.go:2694]  |      127.0.0.1| 200 |    3.05153ms|   match| GET      /index   r:/index
# 浏览器显示
你好
```
此时,调用都是正常的.可以多试几次都没有问题.首先调用请求方法输出"start...",然后远程接口返回,然后当前接口返回
##### 超时情况
模拟超时情况,将参数to设为1,`http://localhost:8080/index?to=1&err=0`,查看结果:
```
start exec.........
error:  hystrix: timeout
OPEN:  false ALLOW:  true
2018/08/30 14:22:10.635 [D] [server.go:2694]  |      127.0.0.1| 200 |10.001375723s|   match| GET      /demo   r:/demo
# 浏览器显示
401
```
* 首先调用执行方法.输出 start...
* 当过了配置的超时时间后,不再等待远程接口返回,返回错误.然后输出当前break的状态.还没有断路.虽然最后远程接口返回了,但是没有作用

一次调用,超时,并没有断路,因为这属于返回错误,而默认设置在调用20次后才开始统计错误.现在修改下配置,然后重启:
```
var config = hystrix.CommandConfig{

    Timeout: 1000, //超时时间,调用接口最多等待时间,超时了,立马返回,不等待接口  单位毫秒

    MaxConcurrentRequests: 10, //最大请求数.同时调用这个接口的最大线程数

    SleepWindow: 5000, //当断路后,等待多长时间，熔断器会再次放行一个调用线程去验证是否恢复。单位毫秒

    ErrorPercentThreshold: 10, //错误率,当远程接口返回错误的数量/总调用次数,大于这个百分比,会断路.比如调用100次,有十次以上是错误

    RequestVolumeThreshold: 2, //请求阈值  熔断器是否打开首先要满足这个条件；这里的设置表示至少有5个请求才进行ErrorPercentThreshold错误百分比计算
}
```
* 修改ErrorPercentThreshold: 为10,也就是10次中有一次错误就断路
* 修改RequestVolumeThreshold 为2,也就是从第二次请求开始就统计错误数量

此时再多调用几次,前几次和上面返回相同,单后来几次控制台会输出:
```
error:  hystrix: circuit open
OPEN:  true ALLOW:  false
```
可以看到,hystrix直接返回错误,没有打印start...,说明run方法断路后就不再调用了,Open为true表明断路器打开了,也就是断路了

当过5秒后,再次调用,会发现run方法会执行一次.这是过了sleepWindow时间,如果这次调用成功了,说明远程端口恢复了,断路器关闭,以后的请求可以
正常调用了
##### 错误情况
模拟出错情况,将参数设置为to=0&err=1,此时调用接口,远程接口返回401状态码,当前接口直接返回了error,此时会造成和上面超时一样的
效果,也就是超过10%的错误,就会打开断路器,当过了SleepWindow时间,会尝试检查一下,如果正常了,断路器闭合,以后可以正常访问

### 总结
使用hystrix-go目前测试出现的问题主要在于hystrix.Flush()方法,breaker.IsOpen()和breaker.AllowRequest()方法.
Flush方法会清理当前hystrix中的所有breaker和统计数据,IsOpen()和breaker.AllowRequest()方法,当在执行hystrix.Do()等方法的前面
调用,会导致断路器不能自动检测闭合的问题.建议一般情况下不要调用这三个方法.