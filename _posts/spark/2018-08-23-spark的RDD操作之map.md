---
layout: blog
istop: false
title: "spark的RDD操作之map"
date:  2018-08-23
category: spark
tags:
- 大数据
- spark
---
# sparkRDD操作之map
## map定义
```scala
def map[U: ClassTag](f: T => U): RDD[U]
```
* T: 调用map的Rdd的内容范型,比如`RDD[String]`,T即为String
* U: 调用map的参数函数f的返回值类型
* 返回一个新的RDD,元素类型为U

map的参数为一个函数f,map通过遍历rdd中的每个元素,并将每个元素传给f作为参数,并调用f,最后将f的返回值放入新的rdd中
## 举例
```
val conf = new SparkConf().setAppName("Simple Application").setMaster("local") //
val sc = new SparkContext(conf)

def main(args: Array[String]): Unit = {
val conf = new SparkConf().setAppName("Simple Application").setMaster("local") //
val sc = new SparkContext(conf)

val rdd = sc.parallelize(Seq("a","b","c")) //定义一个新的Rdd,内容为"a","b","c",包含三个元素
val NRdd = rdd.map(toMap) //将rdd中的每个元素传给toMap函数,并将toMap函数返回值,放到新的rdd中
//val NRdd = rdd.map(v => v + "--") //lamda表达式写法

NRdd.foreach(v => println(v))
}
def toMap(v: String): String = {
    v + "--"
}
```
输出结果
```
a--
b--
c--
```
* 每个Rdd可以理解为一个list,如上rdd可以理解为一个包含三个元素的list
* map相当于foreach一下rdd这个list,对每个元素调用一下toMap函数
* map每次调用toMap,将toMap函数的返回值放到新的Nrdd中,可以理解为往list中添加元素
* 最后返回的新Nrdd,包含了所有toMap函数的返回值