---
layout: blog
istop: true
title: "spark读取hdfs中的数据"
background-image: https://o243f9mnq.qnssl.com/2017/06/116099051.jpg
date:  2018-08-17
category: 大数据
tags:
- 大数据
- spark
- hdfs
- hadoop
---

# spark 读取hdfs中的数据
```
import org.apache.hadoop.io.{LongWritable, Text}
import org.apache.hadoop.mapred.TextInputFormat


val conf = new SparkConf().setAppName("Similar_car")
val sc = new SparkContext(conf)
    //hdfs输入路径
    val srcTextRDD = sc.hadoopFile[LongWritable, Text, TextInputFormat](INPUT_PATH + dt)
```
* 注意导包,容易导错