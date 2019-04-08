# ZooKeeper解决了哪些问题
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/zmczmckkk/Distributed-System-Study/pulls)
[![GitHub stars](https://img.shields.io/github/stars/zmczmckkk/Distributed-System-Study.svg?style=social&label=Stars)](https://github.com/zmczmckkk/Distributed-System-Study)
[![GitHub forks](https://img.shields.io/github/forks/zmczmckkk/Distributed-System-Study.svg?style=social&label=Fork)](https://github.com/zmczmckkk/Distributed-System-Study)
          
　　搬运来源:
   [https://www.cnblogs.com/likehua/p/3999600.html](https://www.cnblogs.com/likehua/p/3999600.html)
   　　作者:李克华  
   　　本人只做板式调整，如有冒犯，请联系。
## 问题导读
  1. master挂机，传统做法备份必然是以前数据，该如何保证挂机数据与备份数据一致？
  2. 分布式系统如何实现对同一资源的访问，保证数据的强一致性？
  3. 集群中的worker挂了，传统做法是什么？zookeeper又是如何做的？  
  
　　分布式系统的运行是很复杂的，因为涉及到了网络通信还有节点失效等不可控的情况。下面介绍在最传统的master-workers模型，主要可以会遇到什么问题，传统方法是怎么解决以及怎么用zookeeper解决。

### Master节点管理
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
### 配置文件管理
　　集群中配置文件的更新和同步是很频繁的，传统的配置文件分发都是需要把配置文件数据分发到每台worker上，然后进行worker的reload，这种方式是最笨的方式，结构很难维护，因为如果集群当中有可能很多种应用的配置文件要同步，而且效率很低，集群规模一大负载很高。还有一种就是每次更新把配置文件单独保存到一个数据库里面，然后worker端定期pull数据，这种方式就是数据及时性得不到同步。
```
    解决问题:   
    1. 统一配置文件分发并且及时让worker生效
```
### 分布式锁
　　在一台机器上要多个进程或者多个线程操作同一资源比较简单，因为可以有大量的状态信息或者日志信息提供保证，比如两个A和B进程同时写一个文件，加锁就可以实现。但是分布式系统怎么办？需要一个三方的分配锁的机制，几百台worker都对同一个网络中的文件写操作，怎么协同？还有怎么保证高效的运行？
```
    解决问题:   
    1. 高效分布式的分布式锁
```
### 分布式锁
　　在一台机器上要多个进程或者多个线程操作同一资源比较简单，因为可以有大量的状态信息或者日志信息提供保证，比如两个A和B进程同时写一个文件，加锁就可以实现。但是分布式系统怎么办？需要一个三方的分配锁的机制，几百台worker都对同一个网络中的文件写操作，怎么协同？还有怎么保证高效的运行？
* zookeeper分布式锁

　　分布式锁主要得益于ZooKeeper为我们保证了数据的强一致性，zookeeper的znode节点创建的唯一性和递增性能保证所有来抢锁的worker的原子性。
```
    解决问题:   
    1. 高效分布式的分布式锁
```
### 集群worker管理
　　集群中的worker挂了是很可能的，一旦workerA挂了，如果存在其余的workers互相之间需要通信，那么workers必须尽快更新自己的hosts列表，把挂了的worker剔除，从而不在和它通信，而Master要做的是把挂了worker上的作业调度到其他的worker上。同样的，这台worker重新恢复正常了，要通知其他的workers更新hosts列表。传统的作法都是有专门的监控系统，通过不断去发心跳包(比如ping)来发现worker是否alive，缺陷就是及时性问题，不能应用于在线率要求较高的场景
zookeeper监控集群

* zookeeper监控集群  
　　利用zookeeper建立znode的强一致性，可以用于那种对集群中机器状态，机器在线率有较高要求的场景，能够快速对集群中机器变化作出响应。
```
    解决问题:   
    1. 集群worker监控
```
