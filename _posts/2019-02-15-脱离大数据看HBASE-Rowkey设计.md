---
layout:     post
title:      脱离大数据看HBASE-Rowkey设计
subtitle:   HBase
date:       2019-02-19
author:     TBKK
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - JAVA
---


# 什么是Rowkey

* 类似于 MySQL、Oracle中的主键，用于标示唯一的行
* 完全是由用户指定的一串不重复的字符串；
* HBase 中的数据永远是根据 Rowkey 的字典排序来排序的。



# 总结
Rowkey的设计有多种方式，主要看业务场景，不同的业务场景，使用合适的设计可以事半功倍。
 


 

