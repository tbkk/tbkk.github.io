---
layout:     post
title:      深入理解Zookeeper
subtitle:   zookeeper
date:       2019-09-15
author:     TBKK
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - JAVA
---


# 什么是Zookeeper  
我们理解的做多的zk都是以注册中心来了解的。比如eureka，consol，etcd，nacos等。
实际上zk应该算是一个分布式系统的协调器。
你们可能还不了解 Zookeeper实际上是Hadoop项目中的一个子项目。

# ZK的角色分类
## leader
* 一个集群中只有1个leader
* leader负责写操作，然后广播到其他节点，如果有超过一半的写入成功就表示成功了
## follower
* 集群中有多个follower
* 负责读数据，如果有写数据，则发送给leader，由leader来写
* 当leader挂了负责投票产生新的leader
## observer
* 没有投票权的follower，增加了扩展性，同事也没有增加投票时间。

# 选举算法
目前最新的选举算法是基于TCP的FastLeaderElection算法。
## myid
每个server节点都有一个id，在配置文件中存储。表明唯一ID
## zxid
类似于RDBMS中的事务ID，用于标识一次更新操作的Proposal ID。为了保证顺序性，该zkid必须单调递增。
因此ZooKeeper使用一个64位的数来表示，高32位是Leader的epoch，从1开始，每次选出新的Leader，epoch加一。低32位为该epoch内的序号，每次epoch变化，都将低32位的序号重置。这样保证了zkid的全局递增性。

## 服务器状态
* LOOKING ：找不到leader，会发起选举
* FOLLOWING： 跟随者
* LEADING： 领导者，唯一的负责写的节点
* OBSERVING： 观察者状态。不参与选举的跟随者


## 选票结构
* logicClock ： 每个服务器会维护一个自增的整数，名为logicClock，它表示这是该服务器发起的第多少轮投票
* state ： 当前服务器的状态
* self_id ： 当前服务器的myid
* self_zxid ： 当前服务器上所保存的数据的最大zxid
* vote_id ： 被推举的服务器的myid
* vote_zxid ： 被推举的服务器上所保存的数据的最大zxid

## 选票PK
外部投票的logicClock大于自己的logicClock，则将自己的logicClock及自己的选票的logicClock变更为收到的logicClock

若logicClock一致，则对比二者的vote_zxid，若外部投票的vote_zxid比较大，则将自己的票中的vote_zxid与vote_myid更新为收到的票中的vote_zxid与vote_myid并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。如果票箱内已存在(self_myid, self_zxid)相同的选票，则直接覆盖

若二者vote_zxid一致，则比较二者的vote_myid，若外部投票的vote_myid比较大，则将自己的票中的vote_myid更新为收到的票中的vote_myid并广播出去，另外将收到的票及自己更新后的票放入自己的票箱

即优先级为logicClock > vote_zxid > vote_myid

## 选举逻辑
org.apache.zookeeper.server.quorum.FastLeaderElection




# Zookeeper怎么防止脑裂的

采用过半机制，及时分成2个集体了，还是需要过半的人投票才能通过。

这里的过半数量指的是，集群正常情况下应有的服务器数量，而不是脑裂后的各自的集群服务器数量，即说明，每个服务器都知道总服务器数量。
请看如下代码：

``` java
public QuorumMaj(Map<Long, QuorumServer> allMembers) {
    this.allMembers = allMembers;
    for (QuorumServer qs : allMembers.values()) {
        if (qs.type == LearnerType.PARTICIPANT) {
            votingMembers.put(Long.valueOf(qs.id), qs);
        } else {
            observingMembers.put(Long.valueOf(qs.id), qs);
        }
    }
    half = votingMembers.size() / 2;
}
 
public boolean containsQuorum(Set<Long> ackSet) {
    return (ackSet.size() > half);
}
``` 

# 总结
技术最终为业务服务的，最终为业务场景提供支撑。一定要举一反三，活学活用，不要拿来主义。
 

