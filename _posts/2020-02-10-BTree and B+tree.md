---
layout:     post
title:      简述B-Tree和B+Tree
subtitle:   数据库索引
date:       2020-2-10
author:     TBKK
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - 数据库
---

# B-Tree和B+Tree的区别

## B-Tree
![image](http://www.qinxinfeng.com/img/others/B-Tree.jpg)

1. 每个节点都有数据
2. 数据和数据之间没有指正
3. 单个查询效率高，不方便遍历


## B+Tree
![image](http://www.qinxinfeng.com/img/others/B+Tree.jpg)

1. 只有叶子节点有数据
2. 数据和数据之间有指正
3. 单个查询效率稳定，方便遍历

## 为什么Mysql用B+Tree，而Mongodb用B-Tree
B+Tree适合遍历，join的时候需要用到，mongodb适合单个查询，如果要关系查询不符合设计初衷。


