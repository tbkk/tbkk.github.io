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

* 类似于 MySQL、Oracle中的主键，用于标识唯一的行
* 完全是由用户指定的一串不重复的字符串；
* HBase 中的数据永远是根据 Rowkey 的字典排序来排序的。

# Rowkey的作用

* 读写数据时通过 RowKey 找到对应的 Region；
* MemStore 中的数据按 RowKey 字典顺序排序；
* HFile 中的数据按 RowKey 字典顺序排序。

# RowKey对查询的影响

以 userId+phone+name 作为rowkey

支持场景：
* userId = 111 AND phone = 158 AND name = tbkk
* userId = 111 AND phone = 139
* userId = 111 AND phone = 13xxxxxx
* userId = 111

难以支持场景：
* phone = 158 AND name = tbkk
* phone = 139
* name = tbkk

# RowKey对Region的影响
* HBase 表的数据是按照 Rowkey 来分散到不同 Region，不合理的 Rowkey 设计会导致热点问题。热点问题是大量的 Client 直接访问集群的一个或极少数个节点，而集群中的其他节点却处于相对空闲状态。
* 如果电话号码作为RowKey，则很容易产出热点

# RowKey设计技巧
## Salting
Salting 的原理是将固定长度的随机数放在行键的起始处
![image](http://www.qinxinfeng.com/img/hbase/13.jpg)

* 优缺点：由于前缀是随机生成的，因而如果想要按照字典顺序找到这些行，则需要做更多的工作。从这个角度上看，salting增加了写操作的吞吐量，却也增大了读操作的开销

## Hashing
Hashing 的原理将RowKey进行hash计算，然后取hash的部分字串和原来的RowKey进行拼接
![image](http://www.qinxinfeng.com/img/hbase/14.jpg)

* 优缺点：可以一定程度打散整个数据集，但是不利于Scan；由于不同数据的hash值可能一样，实际应用中可以使用md5计算，然后截取前几位的字符串。如下：subString(MD5(设备ID), 0, x) + 设备ID，其中x一般取5或6。

## Reversing
Reversing 的原理是反转一段固定长度或者全部的键
![image](http://www.qinxinfeng.com/img/hbase/15.jpg)

* 优缺点：有效地打乱了行键，但是却牺牲了行排序的属性和可读性,并且不是所有的都合适，个人比较喜欢这种方式，毕竟Rowkey也是需要存储空间的，这种方式节约了空间。


# 总结
Rowkey的设计有多种方式，主要看业务场景，不同的业务场景，使用合适的设计可以事半功倍。
 


 

