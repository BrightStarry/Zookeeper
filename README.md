###  Zookeeper 
高效的分布式协调服务。基于Paxos算法，(也就是选举Slave的算法，这种算法总节点数为奇数比较好)。
还有ZAB协议
可以解决分布式一致性的问题。

从设计模式角度看，是一个基于观察者模式设计的分布式服务管理框架，负责存储和管理大家都关心的数据，
然后接收观察者的注册，一旦这些数据的状态发生比那话，zookeeper就负责通知已经在zookeeper上注册的
那些观察者作出相应的反应，从而实现集群中类似Master/Slave管理模式。

* 配置管理:例如机器的配置列表，运行时的开关配置，数据库配置信息等。
一般都数据量比较小，数据内容运行时动态变化，集群中各节点共享信息，配置一致。
* 集群管理：选举出Leader管理集群
* 发布与订阅：分布式数据的分布和订阅。
* 数据库切换：例如初始化zookeeper时读取节点上的数据库配置文件，一旦配置发生变化，它能帮助我们
将变更的通知发到各个客户端，每个客户端收到这个通知后，可以获取最新的数据。
* 分布式日志收集：收集集群中的所有日志信息，进行统一管理。
* 分布式锁
* 队列管理

zookeeper原生API实现分布式功能非常困难。可以使用Curator框架等。

---
* 顺序一致性：一个客户端发起的事务请求，最终会严格地按照其发起的顺序被应用到zookeeper中去。
* 原子性：所有事务请求的处理结果在整个集群的所有机器上的应用情况是一致的；也就是说，要么
整个集群所有的机器都成功应用了某一事务，要么都没有应用。
* 单一视图：无论客户端连接的是哪一个zookeeper服务器，看到的服务端数据都是一致的。
* 实时性:一旦事务被应用。客户端就会立刻从服务器上获取变更后的数据。zookeeper仅能保证一段时间
内，客户端一定能从服务端读取最新的数据状态。
---
1. 简单的数据结构，树形结构。（树形名字空间）
2. 可以构建集群。一般用3-5台机器就可以组成一个zookeeper集群。一半以上的节点挂了，就不对外提供服务了。
3. 对于每个客户端的每个请求，zookeeper都会分配一个全局唯一的递增编号，该编号反映了所有事务的先后顺序。
4. 高性能，zookeeper将全量数据存储在内存中，并直接服务于所有的非事务请求，因此在度操作下性能突出。
---
Zookeeper组成，分为Leader(领导者)、Follower(追随者)、Observer(观察者)，follower和observer被统称为learner（学习者）
* Leader: 负责客户端的Writer请求;
* Follower:负责客户端的reader请求，参与leader选举等；
* Observer：特殊的Follower，可以接受客户端的reader请求，但不参与选举。（扩容系统的支撑能力，提高读取
速度。因为它不支持任何同步的写入请求，只负责和leader同步数据）

---
---
### Zookeeper 安装 
1. 解压
    tar -zxvf zookeeper-3.4.5.tar.gz 
    
2. 修改环境变量，使用:分隔
    vim /etc/profile
    在尾部增加如下：
    export ZOOKEEPER_HOME=/zx/zookeeper/
    export PATH=$ZOOKEEPER_HOME/bin:$PATH
    export PATH
    刷新
    source /etc/profile
    
3. 修改zookeeper的配置文件
    cd /zx/zookeeper/conf
    mv zoo_sample.cfg zoo.cfg
    vim /zx/zookeeper/conf/zoo.cfg
    修改如下：
    dataDir=/zx/zookeeper/data #数据路径
    dataLogDir=/zx/zookeeper/logs #日志路径
    在最后追加   #ip根据需要修改，端口号就是:2888:3888
    server.0=192.168.2.104:2888:3888  
    server.1=192.168.2.105:2888:3888
    server.2=192.168.2.106:2888:3888
    
    2888这个端口表示这个服务器与集群中的Leader服务器交换信息的端口。
    3888这个端口表示，Leaser挂了后，需要通过这个端口进行重新选举。
    
    #创建文件夹
    mkdir /zx/zookeeper/logs -p 
    mkdir /zx/zookeeper/data -p 
    vim /zx/zookeeper/data/myid #创建文件myid，分别输入 0   1    2  （每个文件就这么一个数字就可以了）

4. 启动
    因为已经配置了环境变量
    zkServer.sh start  #启动，也可以查看logs/xxx.out启动日志
    zkServer.sh restart #重启
    zkServer.sh stop #停止
    zkServer.sh status #状态
    
    zkCli.sh #进入客户端
    
 2017年5月13日 13:42:32
 第一次配置遇到一个bug，一直不知道什么原因。我直接把所有虚拟机删了。。重新安装，才好。很难受。
 ！！！注意，成功的这次，我没有设置hosts。
---
#### Zookeeper 客户端命令
查找 ls
创建并赋值 create /zx xxx
获取  get /zx
设值  set /zx xxx
递归删除节点 rmr /path
删除指定某个节点 delete /path/child
---
#####eclipse插件
zookeeperBrowser
#####idea插件
搜索zookeeper
安装，然后进入setting --》 other setting --》 zookeeper,设置ip和端口等就可以了

---
### Java操作zookeeper
有原生的api：zookeeper.jar
还有zkClient.jar,是在原生API基础上扩展的。
上面都不太好用，
最好用的是Curator API。
---
#### zookeeper API  见bjsxt.zookeeper包,ZookeeperBase类
http://blog.csdn.net/liu88010988/article/details/51577783 #api基本操作
ZooKeeper这个类的构造方法可以的参数
connectString : 连接服务器地址列表，用 "," 分隔
sessionTimeout :心跳检测时间间隔，毫秒
watcher:事件处理通知器
canBeReadOnly：标识当期会话是否支持只读
sessionId和sessionPasswd：连接服务器的帐号密码
---
注意，这个类的初始化，也就是连接过程是一个异步的过程。如果连接过慢，代码继续往下执行，
可能连接还没有建立就执行了其他代码，所以需要使用CountDownLatch进行阻塞。
---
Zookeeper API实现分布式锁
zookeeper可以创建一个临时节点（这个节点的概念好像就是一个文件），这个临时节点只对本次会话有效。
那么当分布式项目中的一个节点想对User表的id为1001的数据进行操作，就先获取一次/user/1001这个节点，如果没有，那么自己就创建一个。
那么此时，其他节点如果也想修改这个1001，就要去获取一次/user/1001这个节点，就会发现已经存在，就不予修改。直到前一个会话结束，临时节点
被删除。
---
创建节点（znode）方法：create：
有同步创建，异步创建两种：
同步方式：
    参数1，节点路径（名称）;/x  不允许递归创建，也就是父节点不存在，不允许创建子节点。
    参数2，节点内容，byte[]型。不支持序列化方式，如果要实现序列化，可使用hession、kryo等
    参数3，节点权限，使用Ids.OPEN_ACL_UNSAFE即可。这个参数在一般情况下，无需关注。
    参数4，节点类型，有四种：CreateMode.PERSISTENT(持久节点),PERSISTENT_SEQUENTIAL(持久顺序节点)
    EPHEMERAL(临时节点),EPHEMERAL_SEQUENTIAL(临时顺序节点)
异步方式：在同步的基础上多了两个参数
    参数1，注册一个异步回调函数，需要实现AsyncCallBack.StringCallBack接口，
    重写processResult(int rc,String path,Object ctx,String name)方法，节点创建完毕后将执行该方法。
    rc为状态码(0:成功，-4:端口连接，-110：指定节点存在,-112：会话过期),path：节点路径，name：节点实际名称
    参数2，传给回调方法的参数，也就是上面的ctx
---  
zookeeper API中有watch事件，是一次性触发的（触发后，下次不再触发）。当watch监听的数据(节点)发生变化时，会触发watcher。
会传递对应的事件类型和状态类型，和节点路径，但不会说明变化的值
事件类型（znode节点相关的）：EventType.NodeCreated/NodeDataChanged/NodeChildrenChanged/NodeDeleted
状态类型(跟客户端实例相关的)：KeeperState.Disconnected/SyncConnected/AuthFailed/Expired
可以在修改、增加、判断是否存在节点等各种操作时，设置watch，当然，都只有一次性。
---
ACL(AccessControlList)（AUTH）权限控制。
权限模式：
    ip：通过ip地址粒度来进行权限控制。也可以根据网段的粒度。
    Digest:最常用。就是username和password。见auth包
    World：开放的权限控制模式。可以看作特殊的Digest。仅仅是一个标识。
    Super:超级用户模式，可以对zookeeper进行任何操作。
权限对象：指权限赋予的用户或一个指定的实体。例如ip和机器等。
权限：指通过权限检测后被允许的操作，分为五大类
    CREATE/DELETE/READ/WRITE/ADMIN
---
---
#### zkClient API 封装了原生API  详见zkclient包
创建节点时可以递归创建、删除。
对于watch，它使用了subscribeChildChange()方法，不再是一次性的。
参数1，path路径，参数2，实现了IZkDataListener接口的类，
只需要重写其
handlerChildChanges(String parentPath,List<String> currentChilds)方法，
其中，参数parentPath为所监听节点的全路径，currentChilds为最新的子节点列表（相对路径）;
它针对下面三个事件触发：新增子节点、减少子节点、删除节点，不监听改变

还有subscribeDataChange()方法，监听子节点删除和子节点修改。

---
---
#### Curator API 目前最好的 需要导入curator-framework、curator-recipes、curator-client等jar。
可以实现session超时重连、主从选举、分布式计数器、分布式锁等各种复杂zookeeper场景的api封装。

详见curator包

使用链式编程风格，易读性强，使用工厂方法创建连接对象。
1.使用CuratorFrameworkFactory的两个静态工厂方法（参数不同）实现:
    参数1：connectString，连接串,
    参数2:retryPolicy，重试连接策略，四种实现分别为：
    ExponentialBackoffRetry,RetryNTimes,RetryOneTimes,RetryUntilElapsed
    参数3：sessionTimeoutMs，会话超时时间，默认为60 000ms，
    参数4：connectionTimeoutMs，连接超时时间，默认为15 000ms。
---
java.lang.NoSuchMethodError:org.apache.zookeeper.server.quorum.flexible.QuorumMaj.<init>(Ljava/util/Map;，出现这个错误的原因是因为zookeeper服务器的版本与zookeeper.jar的版本不一致，因此将zookeeper.jar升级到与zookeeper服务器对应的3.5.2。再次运行，又报java.lang.NoSuchMethodError: com.google.common.collect.Sets.newConcurrentHashSet()Ljav;，好吧，一看跟之前的错误一样，都是NoSuchMethodError，我猜想应该是guava的版本与zookeeper.jar所依赖的版本不一致（zookeeper.jar依赖io.netty，而io.netty依赖com.google.protobuf » protobuf-java），so，将guava的版本升级到了20.0，运行成功！
java.lang.NoSuchMethodError: org.apache.zookeeper.server.quorum.flexible.QuorumMaj.<init>(Ljava/util/Map;)V
因为本身引入了curator和zookeeper的jar包 所以我一看这个错误应该是curator是用的zookeeper进行编译的时候和我引进的zookeeper的jar不一致。

！！！2017年5月13日 18:36:28
使用了curator 3.x.x 版本和zookeeper 3.5.x 版本，结果创建节点一直不行，其他都可以。
换了 curator 2.x.x 版本和zookeeper 3.4.x 版本，就好了。难受。
--- 
2. 创建节点方法，create()，可选链式
creatingParentIfNeeded,withMode,firPath,withACL
3. 删除节点方法，delete()，可选链式
deletingChildrenNeeded，guaranteed,withVersion,forPath
4. 读取和修改数据getData(),setData()
5. 异步绑定回调方法，（也就是异步增改节点）比如创建节点时绑定一个回调方法，该回调方法可以输出服务
器的状态码（原生API中的rc参数）以及服务器事件类型，还可以加入一个线程池进行优化操作
(每个修改创建操作都可以绑定回调函数，也就是一个线程，所以需要线程池)。
6. 读取字节点方法，getChildren()
7. 判断字节点是否存在方法， checkExists()
---
#####　Curator的Watch(监听)  需要curator-recipes jar
NodeCacheListener :监听节点的新增，修改操作，删除不触发
PathChildrenCacheListener:监听字节点的新增、修改、删除操作(好像只能监听下面一级的子节点)

其实现不再是原生API的一次性watch，而是使用缓存。

---
#####  Curator的分布式锁 详见lock包中的lock2类
分布式场景中，为了保证数据的一致性，在程序运行的某一点需要同步操作。

InterProcessMutex类，可以锁住zookeeper中的一个节点（例如/a）。锁住后，其他机器将无法访问。
那么如果想锁住User表的id为1001的记录，就可以创建/user/1001节点（应该可以是临时节点），然后将它锁住
---
##### Curator的分布式计数器 详见atomicinteger包
DistributedAtomicInteger类

---
##### Curator的Barrier  让多个线程在某个点同步（相互等待） 详见barrier包

DistributedBarrier类，
barrier = new DistributedBarrier(cf, "/super"); //让绑定同一个节点的多个机器同时等待，
直到调用removeBarrier();方法
barrier.setBarrier();	//设置
barrier.waitOnBarrier();	//等待

DistributedDoubleBarrier类，
barrier.enter(); 进入准备阶段，全部线程等待。都执行到这个方法的时候再往下执行
...执行其他代码
barrier.leave(); 退出，所有线程都执行到这个方法的时候，才能往下继续执行

---





 
    
   



