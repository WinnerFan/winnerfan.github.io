# ZooKeeper

## 简介

雅虎创建，分布式协调服务，实现数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁、分布式队列。

- 顺序一致性：同一个客户端按发起事务顺序进入ZooKeeper
- 原子性：整个集群所有机器成功应用某一事务
- 单一视图：客户端连接任一服务器，结果相同
- 可靠性：服务器状态一直保留
- 实时性：立即读取

## 概念

### 集群角色

- Leader：负责客户端**读写**请求，事务唯一调度和处理者，保证事务处理顺序性；集群内部个各服务器的调度者
- Follower：负责客户端**非事务(读)**请求，转发事务(写)请求给Leader；参与事务请求Proposal投票；参与Leader选举
- Observer：特殊的Follower，可以接受客户端**读**请求，转发事务(写)请求给Leader；不参与Proposal投票(过半写成功)，选举。扩容系统支撑、不影响集群事务处理(写)的前提下提高非事务(读)速度

### 节点

- 机器节点
- 数据节点：临时(会话)、持久；SEQUENTIAL自增

## 使用

### 配置文件

- tickTime：服务端与客户端心跳，毫秒
- dataDir：数据及数据日志
- clientPort：客户端连接端口
- initLimit：初始连接可忍受心跳数
- syncLimit：请求应答可忍受心跳数
- server.A=B:C:D：几号服务器、IP、Leader交换信息端口、选举Leader端口

### zk客户端

#### zkCli.sh

- create [-s/e] path data [acl]：创建顺序、临时、缺省为永久节点，acl为权限控制
- ls/get path [watch]
- set path data [version]
- delete path [version]

#### Java API

1. 建立连接

	- connectString：IP:Port
	- sessionTimeout：心跳
	- watcher：时间处理通知器
	- canBeReadOnly：标记只读
	- sessionId、sessionPasswd：确定唯一一台客户端，目的可重复会话

2. 增删改查存

	- create
		- path：路径，父节点不存在不能创建子节点
		- data：节点内容，不支持序列化
		- acl：节点权限，Ids.OPEN_ACL_UNSAFE
		- createMode：节点类型。persistent持久、persistent_sequential持久顺序、ephemeral临时、ephemeral_sequential临时顺序
		- cb：异步回调函数 ，实现AsynCallBack.StringCallBack接口，重写 processResult(int rc, String path, Object ctx, String name) 方法，当节点创建完毕后执行此方法。服务器响应码0成功、-4端口连接、-110节点存在、-112会话过期，path路径，ctx值，节点名称
		- ctx：传给回调函数的参数，一般为上下文
	- getChildren 监听后，子节点被添加或删除，服务器发送一个NodeChildrenChanged通知， 需要客户端主动重新获取，watcher是一次性的
		- path：路径
		- watcher：注册的watcher，可以为null
		- watcher：是否需要watcher
		- cb：回调函数：自定义类实现AsynCallBack.Children2Callback
		- ctx：上下文信息
		- stat：节点状态信息
	- getData
		- path：路径
		- watcher：注册的watcher，可以为null
		- watcher：是否需要watcher
		- cb：回调函数：自定义类实现AsynCallBack.DataCallback
		- ctx：上下文信息
		- stat：节点状态信息
	- setData
		- path：路径
		- data：数据
		- version：-1覆盖之前所有版本
		- cb：回调函数：自定义类实现AsynCallBack.StatCallback
		- ctx：上下文信息
	- delete
		- path：路径 
	- exists 无论节点是否存在，都可以注册watcher，但子节点变化不会通知客户端
		- path：路径
		- watcher：注册的watcher，可以为null
		- cb：回调函数
		- ctx：上下文信息

3. 增加权限
	
	- addAutoInfo(String scheme, byte[] auth)
		- scheme: model world、auth、digest、ip、super
		- 对于删除而言，权限范围为其子节点，可以删除当前已设置权限节点

4. 事件类型，实现public void process(WatchedEvent event)接口

	- 状态类型（客户端实例相关）
		- KeeperState.SyncConnected：-1,1,2,3,4
		- KeeperState.Disconnected：-1
		- KeeperState.AuthFailed：-1
		- KeeperState.Expired：-1
	
	- 事件类型（节点相关）
		- EventType.None：-1，成功建立会话，断开状态，会话超时，授权失败
		- EventType.NodeCreated：1
		- EventType.NodeDeleted：2
		- EventType.NodeDataChanged：3
		- EventType.NodeChildrenChaged：4
	

#### ZkClient

#### Curator

- 事件监听：NodeCache，PathChildrenCache，TreeCache
- Master选举：多台机器同时创建一个子节点，最终只有一台成功创建。LeaderSelector
- 公平可重入分布式锁：InterProcessMutex
- 分布式计数器：DistibutedAtomicInteger
- 分布式Barrier：多线程同步(CyclicBarrier)，DistributedBarrier
- 工具类
	- ZKPaths创建ZNode路径、递归创建删除
	- EnsurePath节点不存在则创建
	- TestingServer启动一个ZooKeeper服务器
	- TestingCluster启动一个ZooKeeper集群

## 场景

### 数据发布/订阅

应用启动拉取zk，并注册watcher 

### 负载均衡

- 域名配置：/DDNS/app1/serviceA.app1.company1.com->ip:port
- 域名解析：应用解决
- 域名变更：同数据发布/订阅
- 域名注册：Register针对应用
- 域名解析：Dispatcher针对消费者
- 域名探测：可用性检测，ZK服务端心跳(TCP长连接)；ZK客户端发起心跳

### 命名服务

UUID(Universally Unique Identifier)，通用唯一标识符，32位字符和4个短线；过长，语义不明。

利用ZK API自带的顺序命名节点，利用完整路径作为全局唯一。

### 分布式协调/通知

MySQL数据复制总线：Core，Server，Monitor。

- core组件，创建/mysql_replicator/tasks/tast_name
- /mysql_replicator/tasks/tast_name/instances注册热备的主机名，为临时顺序节点，小序号优先RUNNING，其余STANDBY
- 热备切换，每个STANDBY机器在/mysql_replicator/tasks/tast_name/instance注册watcher，RUNNING宕机，重新按照小序号优先选举RUNNING
- /mysql_replicator/tasks/tast_name/lastCommit由RUNING记录执行状态
- server组件，将库名、表名、用户名、密码等数据库信息及消费者信息写入/mysql_replicator/tasks/tast_name
- 冷备切换，core进程被分配了组，/mysql_replicator/task_groups/group1/tast_name/instances，每个core遍历Task列表下Instances若无则此机器创建小号节点执行。这个过程中其他core也会创建节点，所以创建节点后判断是否序号最小，不是最小则删除节点，进入下一个Task

### 机器间通信方式

- 心跳检测：PING、长连接(TCP固有心跳检测)，ZK指定节点下创建临时子节点，判断对应客户端机器是否存活
- 工作进度汇报：每个任务客户端在ZK节点下创建临时子节点，可以判断客户端是否存活；工作进度实时回写
- 系统调度：一个控制台，多个客户端按照流程

### 集群管理(日志收集系统)

变化的日志源，变化的收集器

- 收集器：/logs/collector/host1
	- 任务分发：日志源机器分组写入Host
	- 状态汇报：/logs/collector/host1/status
	- 动态分配：全局；局部，负载上报

### Master选举

强一致性，高并发下节点创建全局唯一性。没有则创建；已有则注册watcher，监听存活

### 分布式锁

1. 排他锁(Exclusive Locks, X)：写锁独占锁，对象仅获得锁的事务可以读写

	- 定义锁：节点为一个锁
	- 获取锁：create临时节点，成功则获取锁，失败则注册watcher
	- 释放锁：节点删除后，通知watcher



2. 共享锁(Shared Locks, S)：读锁，对象对加了读锁事务可读，不可写，直至所有读锁释放

	- 定义锁：节点为一个锁
	- 获取锁：多个子节点，/shared_lock/hos1-R-00001，请求类型可以为读或写请求
	- 判断读写顺序
		- 创建临时子节点，并监听父节点
		- 读请求：没有比自己小的节点或比自己小的节点都是读请求，获取共享锁；比自己小的节点中有写请求，则等待
		- 写请求：最小序号，获取锁；自己不是最小序号节点，则等待
		- watcher通知后循环
	- 释放锁

3. 羊群效应：某些客户端断链后，会通知全部未断链客户端watcher，但只有一少部分启动，大部分维持等待。核心逻辑为判断自己是否是最小的节点，故节点关注比自己小的节点即可。判断读写顺序修改为

- 创建临时子节点，getChildren获取已创建子节点列表
- 读请求：有比自己小的节点或比自己小的节点都是读请求，获取共享锁；向比自己序号小的最后一个写请求节点注册watcher
- 写请求：最小序号，获取锁；向比自己序号小的最后一个节点注册watcher

### 分布式队列

1. FIFO(First Input First Output)

- 获取节点列表
- 最小序号则执行；否则等待，向比自己小的最后一个节点注册watcher

2. Barrier

- 注册watcher监控数量
- 子节点到达一定数量统一执行

### 企业级

1. Hadoop中YARN的HA(High Availability)，YARN中RM(Resource Manager)单点问题，竞争写子节点，假死脑裂(Brain-Split)，zk认为RM1挂了，但其认为自己为Active状态，可以加入ACL权限
2. HBase(Hadoop Database)，数据写入强一致性。HMaster监听ZK节点，ZK节点子节点RegionServer服务器。HMaster不直接心跳监听RegionServer，因为管理负担重，自身挂掉，持久化需求
3. Kafka LinkedIn Scala 开源分布式消息系统。Producer生产者，Consumer消费者，Broker服务器，Topic主题跨服务器，Partition一个主题下多个分区，Group消费者组，Offset消息偏移量。

	- Broker /brokers/id/[0...N]
	- Topic /brokers/topics/[topic]/3->2 Broker ID为3的服务器上两个分区
	- Group /consumers/[group_id]/owners/[topic]/[broker_id-patition_id]->[consumer_id]
	- Offset /consumers/[group_id]/offsets/[topic]/[broker_id-patition_id]->[offset_value]
	- Consumer /consumers/[group_id]/ids/[consumer_id]

4. Metamorphosis消息中间件、Dubbo RPC框架、Canal MySQL Binlog增量订阅和消费组件，Otter分布式数据库同步系统

## 内幕

### 节点

- 持久节点
- 持久顺序节点
- 临时节点
- 临时顺序节点


### 版本(乐观锁)

- version节点数据内容版本号
- cversion子节点版本号
- aversion节点ACL版本号

### Watcher

watcher对象存储在客户端WatchManager中，触发watcher事件后，客户端从WatchManager中取出watcher对象

### ACL(Access Control List)

- 权限模式Scheme：IP、Digest("username:password")、World(只有一个Digest"world:anyone")、Super
- 授权对象ID
- 权限Permission：Create(创建子节点)、Delete(删除子节点)、Read(读其和子节点)、Write(其更新)、Admin