# mongodb常用命令
```
#只有副本集有oplog，如果非集群，可以设置replSet
#创建用户  所有用户相关操作都需要在admin下操作
use.admin  
db.createUser({
  user: 'root',  // 用户名
  pwd: '970125',  // 密码
  roles:[{
    role: 'root',  // 角色
    db: 'admin'  // 数据库
  }]
})

db.createUser({
  user: 'sansi',  // 用户名
  pwd: '970125',  // 密码
  roles:[{
    role: 'backup',  // 角色
    db: 'admin'  // 数据库
  },{
    role: 'restore',  // 角色
    db: 'admin'  // 数据库
  }]
})


mongo "mongodb://root:970125@127.0.0.1:27017/test1?authSource=admin"

db.createUser({
  user: 'ycsb',  // 用户名
  pwd: 'ycsb',  // 密码
  roles:[{
    role: 'readWrite',  // 角色
    db: 'ycsb'  // 数据库
  }]
})

Built-In Roles(内置角色)：
    1. 数据库用户角色：read、readWrite;
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
    4. 备份恢复角色：backup、restore；
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
    6. 超级用户角色：root  
    // 这里还有几个角色间接或直接提供了系统超级用户的访问(dbOwner 、userAdmin、userAdminAnyDatabase)
    7. 内部角色：__system

具体角色的功能： 

Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
#查看mongo集合对应的文件
db.collection_name.stats({indexDetails:true})


db.getReplicationInfo()            #查看 oplog 的状态，输出信息包括 oplog 日志大小，操作日志记录的起始时间

repset:PRIMARY> db.printReplicationInfo()          #查看oplog的状态、总大小、使用大小、存储的时间范围、记录时长。

repset:PRIMARY> db.oplog.rs.stats().maxSize     #显示当前的oplog大小 maxSize


#设置mongodb慢日志等级
db.setProfilingLevel( level , slowms ) 
level：
　　0 – 不开启
　　1 – 记录慢命令 (默认为>100ms)
　　2 – 记录所有命令
slowms：大于多少毫秒为慢日志


1.3.1 repl Buffer size的大小：默认是256MB。
hhlshd2:SECONDARY> db.serverStatus().metrics.repl.buffer.maxSizeBytes
NumberLong(268435456)
＊ 这个暂时没找到可以调整的参数。
 
1.3.2 调整从库replWriter 的数量：
参数：replWriterThreadCount  默认是16，从3.2 开始引入，取值范围： 1-256 
hhlshd2:SECONDARY> db.adminCommand({getParameter:1, replWriterThreadCount:true})
{ "replWriterThreadCount" : 16, "ok" : 1 }
 
1.3.3 replApplyBatchSize：批量oplog的条数。 
hhlshd2:SECONDARY> db.adminCommand({getParameter:1, replApplyBatchSize:true})
{ "replApplyBatchSize" : 1, "ok" : 1 }

mongostat
insert query update delete getmore command dirty used flushes vsize  res qrw arw net_in net_out conn    set repl                time
    *0    *0     *0     *0       0     2|0  0.0% 0.0%       0 1.77G 108M 0|0 1|0   469b   37.0k    5 single  PRI May 19 21:17:54.566
    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.77G 108M 0|0 1|0   262b   36.3k    5 single  PRI May 19 21:17:55.566
q    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.77G 108M 0|0 1|0   262b   36.3k    5 single  PRI May 19 21:17:56.567
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.77G 108M 0|0 1|0   263b   36.4k    5 single  PRI May 19 21:17:57.565
    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.77G 108M 0|0 1|0   262b   36.4k    5 single  PRI May 19 21:17:58.565
常用命令格式：
mongostat --host 192.168.1.100:27017 -uroot -p123456 --authenticationDatabase admin
参数说明：
host:指定IP地址和端口，也可以只写IP，然后使用--port参数指定端口号
-u： 如果开启了认证，则需要在其后填写用户名
-p: 不用多少，肯定是密码
--authenticationDatabase：若开启了认证，则需要在此参数后填写认证库（注意是认证上述账号的数据库）
命令输出格式

各字段解释说明：
insert/s : 官方解释是每秒插入数据库的对象数量，如果是slave，则数值前有*,则表示复制集操作
query/s : 每秒的查询操作次数
update/s : 每秒的更新操作次数
delete/s : 每秒的删除操作次数
getmore/s: 每秒查询cursor(游标)时的getmore操作数
command: 每秒执行的命令数，在主从系统中会显示两个值(例如 3|0),分表代表 本地|复制 命令
注： 一秒内执行的命令数比如批量插入，只认为是一条命令（所以意义应该不大）
dirty: 仅仅针对WiredTiger引擎，官网解释是脏数据字节的缓存百分比
used:仅仅针对WiredTiger引擎，官网解释是正在使用中的缓存百分比
flushes:
For WiredTiger引擎：指checkpoint的触发次数在一个轮询间隔期间
For MMAPv1 引擎：每秒执行fsync将数据写入硬盘的次数
注：一般都是0，间断性会是1， 通过计算两个1之间的间隔时间，可以大致了解多长时间flush一次。flush开销是很大的，如果频繁的flush，可能就要找找原因了
vsize： 虚拟内存使用量，单位MB （这是 在mongostat 最后一次调用的总数据）
res:  物理内存使用量，单位MB （这是 在mongostat 最后一次调用的总数据）
注：这个和你用top看到的一样, vsize一般不会有大的变动， res会慢慢的上升，如果res经常突然下降，去查查是否有别的程序狂吃内存。

qr: 客户端等待从MongoDB实例读数据的队列长度
qw：客户端等待从MongoDB实例写入数据的队列长度
ar: 执行读操作的活跃客户端数量
aw: 执行写操作的活客户端数量
注：如果这两个数值很大，那么就是DB被堵住了，DB的处理速度不及请求速度。看看是否有开销很大的慢查询。如果查询一切正常，确实是负载很大，就需要加机器了
netIn:MongoDB实例的网络进流量
netOut：MongoDB实例的网络出流量
注：此两项字段表名网络带宽压力，一般情况下，不会成为瓶颈
conn: 打开连接的总数，是qr,qw,ar,aw的总和
注：MongoDB为每一个连接创建一个线程，线程的创建与释放也会有开销，所以尽量要适当配置连接数的启动参数，maxIncomingConnections，阿里工程师建议在5000以下，基本满足多数场景


#查看状态：级别和时间
db.getProfilingStatus()
#压测
ycsb maven jdk1.8+
 ycsb有几个目录需要注意下：
bin：
    - 目录下有个可执行的ycsb文件，是个python脚本，是用户操作的命令行接口。ycsb主逻辑是：解析命令行、设置java环境，加载java-libs，封装成可以执行的java命令，并执行

workloads：
    - 目录下有各种workload的模板，可以基于workload模板进行个性化修改

core：
    - 包含ycsb里各种核心实现，比如DB的虚拟类DB.java，各个db子类都要继承该类；还有比如workload抽象类，如果我们要自定义workload实现也需要继承该类

各种DB的目录：
    - 比如mongo，redis等，里面包含了对应测试的源码等。
    - 当ycsb mvn编译后，会在对应的目录下生成target文件，ycsb会加载对应target文件中的class类
2 使用
ycsb在执行的时候，分为两阶段：load阶段 和 transaction阶段

2.1 load阶段
该阶段主要用于构造测试数据，ycsb会基于参数设定，往db里面构造测试需要的数据，如：

mongodb-async
在ycsb中，对于不同的db都有一些选项，比如mongo就有mongodb 和 mongodb-async。 
默 认的mongodb表示同步，即load和run使用同步的方式，ycsb会调用mongodb/src底下对应的MongodbClient实现对应的 insert/update等操作。如果设置了mongodb-async，ycsb会调用mongodb/src底下对应的 AsyncMongoDbClient.java实现

参数设置：
Options:
    -P file        Specify workload file // workload文件
    -cp path       Additional Java classpath entries
    -jvm-args args Additional arguments to the JVM
    -p key=value   Override workload property // 一些设置
    -s             Print status to stderr // 把状态达到stderr中
    -target n      Target ops/sec (default: unthrottled) // 每秒总共操作的次数
    -threads n     Number of client threads (default: 1) // 客户端线程数
参数解读：

-P workload文件
在ycsb的目录下有多种workload，参考：https://github.com/brianfrankcooper/YCSB/wiki/Core-Workloads，我们以workloada举例子 


基础配置：
#YCSB load(加载元数据)命令的参数，表示加载的记录条数，也就是可用于测试的总记录数。
recordcount=1000000
#YCSB run(运行压力测试)命令的参数，测试的总次数。和maxexecutiontime参数有冲突,谁先完成就中断测试退出,结合下面的增改读扫参数,按比例来操作.
#operationcount=1000000
#测试的总时长,单位为秒,和operationcount参数有冲突,谁先完成就中断测试退出,这里3600秒就代表一小时,也是结合下面的增改读扫参数,按比例来操作.
maxexecutiontime=3600
#java相关参数,指定了workload的实现类为 com.yahoo.ycsb.workloads.CoreWorkload
workload=com.yahoo.ycsb.workloads.CoreWorkload
#表示查询时是否读取记录的所有字段
readallfields=true
#表示读操作的比例，该场景为0.54,即54%
readproportion=0.54
#表示更新操作的比例，该场景为0.2,即20%
updateproportion=0.2
#表示扫描操作的比例,即1%,通常是用来模拟没有索引的查询
scanproportion=0.01
#表示插入操作的比例,即25%
insertproportion=0.25
#表示请求的分布模式，YCSB提供uniform(随机均衡读写),zipfian(偏重读写少量热数据),latest(偏重读写最新数据)三种分布模式
requestdistribution=uniform
#插入文档的顺序方式：哈希(hashed)/有序 (insertorder)
insertorder=hashed
#测试数据的每一个字段的长度,单位字节
fieldlength=2000
#测试数据的每条记录的字段数,也就是说一条记录的总长度是2000*10=20000字节
fieldcount=10
#####################以下是不自带的参数#############################
#把数据库连接地址写到这里,那么输命令的时候就不用输地址了,我这里没有设用户名密码,当然是不需要了
mongodb.url=mongodb://10.21.1.205:30000/ycsb?w=0
#如果你是有很多个mongos的话,就要这样写
#mongodb.url=mongodb://172.25.31.101:30000,172.25.31.102:30000,172.25.31.103:30000/ycsb?w=0
#如果是有用户名密码的话,就要这样写,还记得上面一开始的模板嘛,mongodb://user:pwd@server1.example.com:9999
#mongodb.url=mongodb://ycsbtest:ycsbtest@10.21.1.205:30000/ycsb?w=0
#写安全设置,Unacknowledged(不返回用户是否写入成功{w:0}),Acknowledged(写入到主节点内存就返回写入成功{w:1}),Journaled(写入到主节点journal log才返回写入成功{w:1,j:true}),Replica Acknowledged(写入到所有副本集内存和主节点journal log才返回写入成功{w:2}),Available Write Concern(写入到所有节点journal log才返回写入成功{w:2,j:true}),选项越往后,安全性越高,但是性能越差.
mongodb.writeConcern=acknowledged
#线程数,也就是说100个并发,既然是压测,怎么可能是单线程
threadcount=100
 
requestdistribution=zipfian
workloada的负载比较中，read和update类比例为1:1，里面一些设置参数如上，如果我们再设置mongo的时候，还需要再workload中增加对应的mongo配置，如下：

mongodb.url=mongodb://192.168.137.10:34001/ycsb?  # mongodb对应的uri等
mongodb.database=ycsb # 对应的db
mongodb.writeConcern=normal # 写级别
-p选项
-p用于设置一些对应的参数，如果workload中的参数，也可以以-p的方式放在命令行中设置

-s
-s是表示，在运行中，把一些状态打印到stderr中，一般status信息，用于表示在运行中的一些中间状态（比如当前处理了多少请求，还有多少请求等）

-target n
表示1s中总共的操作次数（各个线程加起来的），如果性能不满足，比如最高性能只有100，你设置了1000，那么ycsb会尽量往这个数目去靠近。默认是不做限制

-thread 线程数
设置ycsb client的并发测试线程数，默认是1，单线程，所以再测试的时候，一定要设置这个选项

2.2 transcation阶段
在2.1load数据结束之后，ycsb就可以进行测试了，也就是transaction阶段。在transaction阶段，会基于workload中的比例设置，和线程参数设置进行db的压测。具体参数如上
#造数
./bin/ycsb load mongodb -P workloads/workloada -p mongodb.url=mongodb://ycsb:ycsb@127.0.0.1:27017/ycsb -s
#压测
./bin/ycsb.sh run mongodb -P workloads/workloada -p mongodb.url=mongodb://ycsb:ycsb@127.0.0.1:27017/ycsb -s

##mongodb备份恢复##
##mongodump/mongorestore
#全备+oplog
mongodump --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin  --oplog --out /mongoback/
mongorestore --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin  --oplogReplay /mongoback/
mongorestore --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --drop --oplogReplay /mongoback/
#备份单个集合(只有全备支持oplog)
mongodump --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin  --db=test --collection=t1 --out /mongoback/
mongorestore --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --db=test --collection=t1 /mongoback/test/t1.bson

##mongoexport/mongoimport
#CSV类型必须指定字段，json不需要
mongoexport --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --db=test --collection=t1 --out /mongoback/backup
mongoimport --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --db=test --collection=t1 --file /mongoback/backup

mongoexport --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --type=csv -f id --db=test --collection=t1 --out /mongoback/back.csv
mongoimport --username=zj --password=970125 --host=127.0.0.1 --port=27017 --authenticationDatabase admin --type=csv --db=test --collection=t1 --headerline --file /mongoback/back.csv

config = {_id: 'single1', members: [
              {_id: 0, host: '127.0.0.1:27017'},
            ]
      }
#mongodb节点操作
db.adminCommand(nameOrDocument)-切换到“ admin”数据库，并运行命令[只需调用db.runCommand(...)]
db.aggregate([pipeline]，{options})-在此数据库上执行无集合聚合；返回一个游标
db.auth(用户名，密码)
db.cloneDatabase(fromhost)-仅适用于MongoDB 4.0及以下版本
db.commandHelp(name)返回命令的帮助
db.copyDatabase(fromdb，todb，fromhost)-仅适用于MongoDB 4.0及以下版本
db.createCollection(name，{size：...，capped：...，max：...})
db.createUser(userDocument)
db.createView(name，viewOn，[{$ operator：{...}}，...]，{viewOptions})
db.currentOp()显示数据库中当前正在执行的操作
db.dropDatabase(writeConcern)
db.dropUser(用户名)
db.eval()-已弃用
db.fsyncLock()将数据刷新到磁盘并锁定服务器以进行备份
db.fsyncUnlock()在db.fsyncLock()之后解锁服务器
db.getCollection(cname)与db ['cname']或db.cname相同
db.getCollectionInfos([filter])-返回一个列表，其中包含数据库集合的名称和选项
db.getCollectionNames()
db.getLastError()-仅返回err msg字符串
db.getLastErrorObj()-返回完整状态对象
db.getLogComponents()
db.getMongo()获取服务器连接对象
db.getMongo()。setSlaveOk()允许在复制从属服务器上进行查询
db.getName()
db.getProfilingLevel()-已弃用
db.getProfilingStatus()-返回是否打开分析并降低阈值
db.getReplicationInfo()
db.getSiblingDB(name)在与此服务器相同的服务器上获取数据库
db.getWriteConcern()-返回用于此db上任何操作的写关注点，如果设置，则从服务器对象继承
db.hostInfo()获取有关服务器主机的详细信息
db.isMaster()检查副本主状态
db.killOp(opid)杀死db中的当前操作
db.listCommands()列出所有的db命令
db.loadServerScripts()加载db.system.js中的所有脚本
db.logout()
db.printCollectionStats()
db.printReplicationInfo()
db.printShardingStatus()
db.printSlaveReplicationInfo()
db.resetError()
db.runCommand(cmdObj)运行数据库命令。如果cmdObj是字符串，则将其转换为{cmdObj：1}
db.serverStatus()
db.setLogLevel(level，<component>)
db.setProfilingLevel(level，slowms)0 =关闭1 =慢2 =全部
db.setVerboseShell(flag)在shell输出中显示其他信息
db.setWriteConcern(<写入关注文档>)-设置写入数据库的写入关注
db.shutdownServer()
db.stats()
db.unsetWriteConcern(<写入关注文档>)-取消对写入数据库的写入关注
db.version()服务器的当前版本
db.watch()-打开数据库的更改流游标，以报告其非系统集合的所有更改。
#mongodb副本集常用操作
rs.status(){replSetGetStatus:1}检查repl set状态
rs.initiate(){replSetInitiate:null}使用默认设置设置初始化
rs.initiate(cfg){replSetInitiate:cfg}使用配置cfg设置初始化
rs.conf()从local.system.replset获取当前配置对象
rs.reconfig(cfg)使用cfg(disconnects)更新正在运行的副本集的配置
rs.add(hostportstr)使用默认属性(disconnects)向集合添加新成员
rs.add(membercfgobj)使用附加属性(disconnects)向集合添加新成员
rs.addArb(hostportstr)添加一个新成员，该成员为arbiterOnly:true(断开连接)
rs.steppown([steppownsecs，catchUpSecs])作为主(断开)降压
rs.sync from(hostportstr)从给定成员进行二次同步
rs.freeze(secs)使节点在指定时间内不符合成为主节点的条件
rs.remove(hostportstr)从副本集中删除主机(断开连接)
rs.slaveOk()允许在辅助节点上查询
rs.printReplicationInfo()检查oplog大小和时间范围
rs.printSlaveReplicationInfo()检查副本集成员和复制延迟
db.isMaster()检查谁是主要用户
#mongodb分片常用操作
sh.addShard(主机)server:port或setname/server:port
sh.addShardToZone(shard，zone)将碎片添加到区域
sh.updateZoneKeyRange(全名，min，max，zone)将给定集合的指定范围指定给区域
sh.disable balancing(coll)在一个集合上禁用平衡
sh.enableballancing(coll)在一个集合上重新启用平衡
sh.enableSharding(dbname)在数据库dbname上启用sharding
sh.getBalancerState()返回是否启用平衡器
sh.isBalancerRunning()如果平衡器在任何mongos上都有工作正在进行，则返回true
sh.move chunk(fullName，find，to)将块移动到“find”到“to”(shard的名称)
sh.removeShardFromZone(碎片，区域)从区域移除碎片
sh.removeangefromzone(全名，最小值，最大值)从任何区域中删除给定集合的范围
sh.shardCollection(全名、键、唯一、选项)分割集合
sh.splitAt(全名，中间)分割中间所在的块
sh.splitFind(全名，find)将find所在的块拆分为中间值
sh.startBalancer()启动平衡器，以便自动平衡块
sh.status()打印群集的概述
sh.stopBalancer()停止平衡器，因此块不会自动平衡
sh.disableAutoSplit()在一个集合上禁用autoSplit
sh.enable autoSplit()在一个集合上重新启用autoSplit
sh.getShouldAutoSplit()返回是否启用autosplit


