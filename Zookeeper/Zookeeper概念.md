#ZooKeeper概念

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/zmczmckkk/Distributed-System-Study/pulls)
[![GitHub stars](https://img.shields.io/github/stars/zmczmckkk/Distributed-System-Study.svg?style=social&label=Stars)](https://github.com/zmczmckkk/Distributed-System-Study)
[![GitHub forks](https://img.shields.io/github/forks/zmczmckkk/Distributed-System-Study.svg?style=social&label=Fork)](https://github.com/zmczmckkk/Distributed-System-Study)
          
　　搬运来源:
   [http://developer.51cto.com/art/201809/583184.htm](http://developer.51cto.com/art/201809/583184.htm)   
　　本人只做板式调整，如有冒犯，请联系。
  ## 什么是 ZooKeeper
  ### ZooKeeper 的由来  
　　下面这段内容摘自《从 Paxos 到 ZooKeeper 》第四章第一节的某段内容，推荐大家阅读一下:  
　　`Zookeeper` 最早起源于雅虎研究院的一个研究小组。在当时，研究人员发现，在雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行`分布式协调`，但是这些系统往往都存在分布式单点问题。  
　　所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。  
　　关于`ZooKeeper`这个项目的名字，其实也有一段趣闻。在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的`Pig`项目)，雅虎的工程师希望给这个项目也取一个动物的名字。  
　　时任研究院的首席科学家 Raghu Ramakrishnan 开玩笑地说：“在这样下去，我们这儿就变成动物园了！”  
　　此话一出，大家纷纷表示就叫动物园管理员吧，因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了。
  而 `Zookeeper` 正好要用来进行分布式环境的协调，于是，`Zookeeper` 的名字也就由此诞生了。
  ### ZooKeeper 概念
* `ZooKeeper`是一个开源的分布式协调服务，`ZooKeeper`框架最初是在“Yahoo!"上构建的，用于以简单而稳健的方式访问他们的应用程序  
* 后来，`Apache ZooKeeper`成为`Hadoop`，`HBase`和其他分布式框架使用的有组织服务的标准。例如`Apache HBase`使用`ZooKeeper`跟踪分布式数据的状态。
* `ZooKeeper`的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的`原语集`，并以一系列简单易用的`接口`提供给用户使用。  
```
        原语：操作系统或计算机网络用语范畴。  
        它是由若干条指令组成的，用于完成一定功能的一个过程。  
        具有不可分割性，即原语的执行必须是连续的，在执行过程中不允许被中断。
```      
* `ZooKeeper` 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。
* `ZooKeeper` 一个最常用的使用场景就是用于担任`服务生产者`和`服务消费者`的`注册中心`。
* `服务生产者`将自己提供的服务注册到 ZooKeeper 中心，`服务的消费者`在进行服务调用的时候先到`ZooKeeper`中查找服务，获取到`服务生产者`的详细信息之后，再去调用`服务生产者`的内容与数据。
　　  
　　如下图所示，在 Dubbo 架构中 ZooKeeper 就担任了注册中心这一角色。  
![角色](Zookeeper/resource/img/zookeeper_characoter.jpg)
## Master节点管理
　　集群当中最重要的是Master，所以一般都会设置一台Master的Backup。  
　　Backup会定期向Master获取Meta信息并且检测Master的存活性，一旦Master挂了，Backup立马启动，接替Master的工作自己成为Master，分布式的情况多种多样，因为涉及到了网络通信的抖动，针对下面的情况:
* Backup检测Master存活性传统的就是定期发包，一旦一定时间段内没有收到响应就判定Master Down了，于是Backup就启动，如果Master其实是没有down，Backup收不到响应或者收到响应延迟的原因是因为网络阻塞
的问题呢？Backup也启动了，这时候集群里就有了两个Master，很有可能部分workers汇报给Master，
* 另一部分workers汇报给后来启动的Backup，这下子服务就全乱了。
Backup是定期同步Master中的meta信息，所以总是滞后的，一旦Master挂了，Backup的信息必然是老的，很有可能会影响集群运行状态。  
```
    解决问题:   
    1. Master节点高可用，并且保证唯一
    2. Meta信息的及时同步
```
## 配置文件管理
　　集群中配置文件的更新和同步是很频繁的，传统的配置文件分发都是需要把配置文件数据分发到每台worker上，然后进行worker的reload，这种方式是最笨的方式，结构很难维护，因为如果集群当中有可能很多种应用的配置文件要同步，而且效率很低，集群规模一大负载很高。还有一种就是每次更新把配置文件单独保存到一个数据库里面，然后worker端定期pull数据，这种方式就是数据及时性得不到同步。
```
    解决问题:   
    1. 统一配置文件分发并且及时让worker生效
```
## 分布式锁
　　在一台机器上要多个进程或者多个线程操作同一资源比较简单，因为可以有大量的状态信息或者日志信息提供保证，比如两个A和B进程同时写一个文件，加锁就可以实现。但是分布式系统怎么办？需要一个三方的分配锁的机制，几百台worker都对同一个网络中的文件写操作，怎么协同？还有怎么保证高效的运行？
* zookeeper分布式锁

　　分布式锁主要得益于ZooKeeper为我们保证了数据的强一致性，zookeeper的znode节点创建的唯一性和递增性能保证所有来抢锁的worker的原子性。
```
    解决问题:   
    1. 高效分布式的分布式锁
```
## 集群worker管理
　　集群中的worker挂了是很可能的，一旦workerA挂了，如果存在其余的workers互相之间需要通信，那么workers必须尽快更新自己的hosts列表，把挂了的worker剔除，从而不在和它通信，而Master要做的是把挂了worker上的作业调度到其他的worker上。同样的，这台worker重新恢复正常了，要通知其他的workers更新hosts列表。传统的作法都是有专门的监控系统，通过不断去发心跳包(比如ping)来发现worker是否alive，缺陷就是及时性问题，不能应用于在线率要求较高的场景
zookeeper监控集群

* zookeeper监控集群  
　　利用zookeeper建立znode的强一致性，可以用于那种对集群中机器状态，机器在线率有较高要求的场景，能够快速对集群中机器变化作出响应。
```
    解决问题:   
    1. 集群worker监控
```
