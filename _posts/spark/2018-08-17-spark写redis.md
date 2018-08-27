---
layout: blog
istop: false
title: "spark写数据到redis"
background-image: https://o243f9mnq.qnssl.com/2017/06/116099051.jpg
date:  2018-08-17
category: 大数据
tags:
- 大数据
- spark
- redis
---

## spark写redis
封装到一个类中
```

import org.apache.commons.pool2.impl.GenericObjectPoolConfig
import redis.clients.jedis.JedisPool
import com.usedcar.similarcar.util.Constants._
object RedisClient extends Serializable {

  val redisTimeout = 30000
  lazy val pool = new JedisPool(new GenericObjectPoolConfig(), REDIS_HOST, REDIS_PORT, redisTimeout)

  lazy val hook = new Thread {
    override def run = {
      println("Execute hook thread: " + this)
      pool.destroy()
    }
  }
  sys.addShutdownHook(hook.run)
}

```
### 使用
```
rdd.foreachPartition(toRedis)

def toRedis(iter: Iterator[(String, String)]) = {
    val jedis = RedisClient.pool.getResource

    iter.foreach(v => {
      jedis.setex(v._1, REDIS_EXPIRE, v._2)
    })
    jedis.close()
  }
```
* Scala中使用关键字lazy来定义惰性变量，实现延迟加载(懒加载)。
惰性变量只能是不可变变量，并且只有在调用惰性变量时，才会去实例化这个变量
* 针对每个partition进行遍历,方便使用redis的多连接
* 