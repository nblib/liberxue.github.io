---
layout: blog
istop: true
title: "spark的RDD操作之flatmap"
description: "spark操作rdd,spark的flatmap操作详解"
date:  2018-08-23
category: spark
tags:
- 大数据
- spark
---

spark操作rdd,spark的flatmap操作详解
***

# sparkRDD操作之flatmap
## 定义
```scala
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]
```
* T: 调用map的Rdd的内容范型,比如`RDD[String]`,T即为String
* TraversableOnce 为一个可遍历的类型
* U: 调用map的参数函数f的返回的可遍历对象的内容类型
* 返回一个新的RDD,元素类型为U

flatmap,和map类似,遍历rdd,将每个元素作用于函数f,和map不同的是,f返回值为一个可遍历对象.最后将f的返回值中的元素取出来加入到新的rdd中


## 举例
```
def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Simple Application").setMaster("local") //
    val sc = new SparkContext(conf)

    val rdd = sc.parallelize(Seq("1_one","2_two","3_three")) //定义一个新的Rdd,内容为"a","b","c",包含三个元素
    val NRdd = rdd.flatMap(toFlatMap) //将rdd中的每个元素传给toFlatMap函数,并将toFlatMap函数返回值(可遍历)中的所有元素取出来,放到新的rdd中
    //val NRdd = rdd.map(v => v.split("_")) //lamda表达式写法

    NRdd.foreach(v => println(v))
}
def toFlatMap(v: String) = {
    val strings = v.split("_")
    strings
}
```
输出结果
```
1
one
2
two
3
three
```
* 每个Rdd可以理解为一个list,如上rdd可以理解为一个包含三个元素的list: ["1_one","2_two","3_three"]
* flatmap相当于foreach一下rdd这个list,对每个元素调用一下toFlatMap函数
    - `toflatMap("1_one")` 返回一个分割后的数组: `["1","one"]`
    - `toflatMap("2_two")` 返回一个分割后的数组: `["2","two"]`
    - `toflatMap("3_three")` 返回一个分割后的数组: `["3","three"]`
* map每次调用toMap,将toMap函数的返回值中的每个元素添加到新的rdd中,比如,刚开始新的rdd为一个空的list[]:
    - 第一次执行后的返回值 `["1","one"]` ,将这个数组中的所有元素取出来,放到rdd中,此时:rdd = ["1","one"]
    - 第二次执行后的返回值 `["2","two"]` ,此时:rdd = ["1","one","2","two"]
    - 第三次执行后的返回值 `["2","two"]` ,此时:rdd = ["1","one","2","two","2","two"]
* 最后返回的新rdd


## 补充

### flatMapValues

#### 定义
```
def flatMapValues[U](f: V => TraversableOnce[U]): RDD[(K, U)]
```
* V:键值对的value
* U: 对每个键值对的value进行处理后,返回一个可遍历的集合,U为集合内的元素类型
* RDD: 最后返回结果为一个键值对RDD,键为之前的RDD的键,值为flatMapValues处理后的集合中的一个元素.
flatMapValues作用于pairRDD(键值对RDD).
> pairRDD  
键值对RDD,当RDD中的元素为包含两个元素的集合时,这个RDD为PairRDD,键为集合的第一个元素,值为第二个元素.比如rdd中有两个数据:[(1,"a"),(2,"b")]
可以看到,每个元素为包含两个元素的元组,这个rdd为一个pairRDD.第一个键值对key为`1`,value为`"a"`,第二个key为`2`,value为`"b"`

#### 示例
```
def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Simple Application").setMaster("local") //
    val sc = new SparkContext(conf)

    val rdd = sc.parallelize(Seq("1_one-yi", "2_two-er", "3_three-san")) //(1)
    val pairRdd = rdd.map(v => {            //(2)
      val strings = v.split("_")
      (strings(0), strings(1))
    })
    val results = pairRdd.flatMapValues(toFlatValues) (3)
    results.foreach(f => println(f))
}

def toFlatValues(v: String) = {
    val strings = v.split("-")
    strings
}
```
1. 第一步,生成一个普通的rdd,包含三个元素: `"1_one-yi", "2_two-er", "3_three-san"`
2. 第二步,对rdd的元素进行分割,每条记录分割为一个元组: `(1,"one-yi"), (2,"two-er"), (3,"three-san")`,即变为一个pairRDD
3. 第三步,对pairRDD执行flatMapValues操作,分为以下步骤:
    - 对所有键值对的值执行map操作,即将`"one-yi","two-er", "three-san"`进行map,每个值分别转变为`("one","yi")  ("two","er") ("three","san")`
    - 对值map之后,此时的rdd为`[(1,("one","yi")),(2,("two","er")),(3,("three","san"))]
    - 此时的rdd为一个key对应一个集合,也就是一个key对应多个value
    - 然后执行flat操作,也就是将key和它对应的集合中的每个元素结合为键值对,比如: `(1,(1,("one","yi")))`,转化为`(1,"one"),(1,"yi")`
    - 最后,生成的结果为: `[(1,"one"),(1,"yi"),(2,"two"),(2,"er"),(3,"three"),(3,"san")]`
    