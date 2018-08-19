---
layout: blog
istop: true
title: "DataFrame 和 RDD 区别"
date:  2018-08-17
category: 大数据
tags:
- 大数据
- spark
---

# DataFrame 和 RDD 区别
* RDD是分布式的 Java对象的集合，比如，RDD[Person]是以Person为类型参数,Person类的内部结构对于RDD而言却是不可知的
* DataFrame是一种以RDD为基础的分布式数据集，也就是分布式的Row对象的集合（每个Row对象代表一行记录），提供了详细的结构信息，也就是我们经常说的模式（schema），Spark SQL可以清楚地知道该数据集中包含哪些列、每列的名称和类型
* 和RDD一样,DataFrame的各种变换操作也采用惰性机制
