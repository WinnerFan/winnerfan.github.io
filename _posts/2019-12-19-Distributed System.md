---
layout: post
title: Distributed System
tags: Distributed System
---

## 特点

硬件或者软件组件分布在不同网络计算机上，之间仅仅通过消息传递进行通信和协调。

- 分布式
- 对等性：无主从，但有副本
- 并发性
- 缺少全局时钟
- 故障总会发生：通信问题，网络分区，三态(成功失败超时)、节点故障

## 理论

- ACID(Atomicity, Consistency, Isolation, Durability)：读未提交、读已提交、可重复读、串行化；脏读(读中改)、可重复读(两次读不同，针对改)、幻读(两次读不同，针对增删)
- CAP(Consistency, Availablility, Partition tolerance)：一致性、可用性、分区容错性最多同时满足两个
- BASE(Basically Available, Soft tate, Eventually consistent)：基本可用(响应时间、功能)、软状态(允许不同节点数据同步存在延时)、最终一致性(特殊的弱一致)

## 一致性算法

协调者(Coordinator)与参与者(Participant)

### 2PC(Two-Phase Commit)

两阶段提交，P先执行返回Yes/No给C，全部Yes后Commit，否则Rollback返回Ack给C

- 同步阻塞，等待其他C返回
- 单点问题，协调者一挂全挂
- 数据不一致，部分Commit
- 无容错机制，一个节点失败则事务失败

### 3PC(Three-Phase Commit)

三阶段提交，C询问收到任何No中止，P执行事务预提交C收到任何No中止，P提交C收到任何No中止
- 降低阻塞范围
- 单点问题后，**数据一致**

### Paxos

提案者(Proposer)和接收者(Acceptor)、学习者(Learner)
- P向多数A发送Prepare(V)，A接受V>AcceptedMaxV，并AcceptedMaxV=V，返回Promise(N,V)
	- P收到过半数Promise(N,V)，发送Accept(N,V)，A接受V>=AcceptedMaxV，返回Accepted(N,V)
- A将批准的提案发给L集群

### ZAB(ZooKeeper Atomic Broadcast)

崩溃恢复和消息广播
- 消息广播：类似于一个2PC，Leader生成事务Proposal，发给其余机器，再收集选票，最后提交事务，通知Follower提交。移除了中断模式，而是Follower抛弃Leader；过半Ack，即可提交；无法解决Leader崩溃。FIFO，Proposal分配全局单调递增唯一ID(ZXID)，按照循序排序处理
- 崩溃恢复：Leader崩溃或者Leader与过半Follower失去联系。确保其他机器**提交被Leader提交的事务Proposal，丢弃跳过的事务Proposal**
	- 为此选举拥有集群中ZXID,myID(服务器)最大的事务Proposal的机器作为Leader。非Observer服务器变为LOOKING，机器向集群其他机器投票(ZXID,myID)；是否是本轮投票、是否来自LOOKING状态机器；修改为(ZXID,myID)大的票重新投票；每次投票判断是否过半机器接受到相同投票；确定Leader，状态为LEADING和FOLLOWING
	- 所有Follower向准Leader发送epoch(ZXID高32位)，准Leader e1=epoch+1，e1发送给Follower；Follower更新epoch，返回ACK(epoch、历史事务集)；拥有最大ZXID的历史事务集作为初始化集合；同步过半Commit

### 共识算法: Raft

假设将军中没有叛军，信使的信息可靠但有可能被暗杀的情况下，将军们如何达成一致性决定。

节点三种状态: **Follower，Candidate，Leader**

- Follower: 起始状态，负责投票，同一term只有一票；选举时间到，向其他节点发送vote，成为Candidate
- Candidate: 选举时间到，term数++，向其他节点发送vote；接收到超过半数的服务器vote，成为Leader，开始发送心跳；接收到Leader心跳或term数大于自身的vote请求，成为Follwer
- Leader: 接收到term数大于自身的vote请求，成为Follwer

1. 日志同步
    - 客户端发送至Leader，Leader保存uncommitted数据
    - Leader同步数据至Follower，Follower保存uncommitted数据
    - Leader收到确认超过半数，uncommited数据变为committed数据，返回给客户端数据
    - Leader发送请求至Follower，Follower数据由uncommited数据变为committed数据

2. 网络隔离日志同步
    - 两个网络合并为一个网络时数据同步问题
    - 第一个网络中Leader有uncommitted数据，第二个网络中Leader全为committed数据
    - 合并网络，数据冲突，第一个网络中Leader降级为Follower，同步为第二个网络中Leader的committed数据


## 领域驱动设计模式(Domain Driven Design)

- 问题空间：业务面临的系列问题和需求，产品
- 解决方案空间：解决方案，技术

- 领域：系统要解决的现实问题，一个对应一个问题空间
- 领域模型：领域里关键事物及其关系的可视化表现，属于解决方案空间

## 事件溯源(Event Sourcing)

- 事件驱动，不保存对象最新状态，而是保存对象产生的所有事件
- 通过事件溯源，得到对象最新状态，事件不会出现DELETE和UPDATE
- 可以将事件聚合出的对象写入一个物化视图这样的表中，做到更新和查询流程隔离

## 读写隔离(Command Query Responsibility Segregation)

- 场景
	- 写模型和读模型差别较大
	- DDD模型实现时，不受ORM框架带来的对象和数据库的阻抗失衡
	- 读/写比非常高
	- 高并发写和高并发读 
- 最终一致性，查询数据可能不是最新的，而是有延迟
- 实现方式
	- 数据库读写分离
	- 存储不分离，上层逻辑代码分离
	- 存储分离，Command类请求，修改数据， web层进入Event Sourcing更新，并有一个监听器Event Handler更新视图，即读库；Query类型请求，web层通过DAO返回视图