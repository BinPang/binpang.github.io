---
layout: post
title:  "计算最近一段时间的总数量"
date:   2023-05-06
categories: [redis]
---

## 背景
最近有一个业务需求：计算最近24小时、最近3天的id数量

## 解决方案

### 方案一
如果使用MySQL，当然很方便，记录数据`id,ctime`, 直接使用`select count(id) from xx where ctime > yy`即可，
显然当查询的`ctime`比较短&数据量比较小时，直接命中覆盖索引会非常快，当时当`ctime`比较长&数据量比较多时会比较慢

### 方案二
我们存储本来一开始就是使用Redis来存储的，最开始计划的是使用`key_20230506`这样的key来存储每天的数据量。
但是这种方式会带来如下2个问题
1. 数据不准(可能多统计0~24小时的数据)
2. 查询多天数据时需要批量获取与求和操作

### 方案三
直接使用Redis的zset来进行存储：`zadd key score member`，使用当前毫秒作为score
计算时使用zcount很方便，例如计算最近3天的总量：`zcount key (now-3d) +inf`。
zcount时间复杂度为`log(n)`是完全足够的

## 总结
当遇到区间问题时，Redis的zset提供了这样的功能，不知道Redis提供了哪些功能，可以把[Redis Commands](https://redis.io/commands/)所有命令翻一遍