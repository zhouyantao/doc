# mongodb 集群运维常用命令
```
一、新增副本集成员
1）登录primary
2）
>use admin
>rs.add(“new_node:port”)
或者
>rs.add({“_id”:4,”host”:”new_node:port”,”priority”:1,”hidden”:false})
3）
>use admin
>rs.addArb(“new_node:port”)
或者
>rs.addArb({“_id”:5,”host”:”new_node:port”})
或者
>rs.add({‘_id’:5,”host”:”new_node:port”,”arbiterOnly”:true})

仲裁者唯一的作用就是参与选举，仲裁者并不保存数据，也不会为客户端提供服务。成员一旦以仲裁者的身份加入副本集中，它就永远只能是仲裁者，
无法将仲裁者重新配置为非仲裁者，反之亦然。最多只能有一个仲裁者每个副本集中。

温馨提示：
如果复制集中 priority=1 （默认），调用rs.add(“new_node:port”) 该命令 会产生 主从切换 即选举操作；
如果复制集中 priority=1 （默认），直接调用rs.remove(“new_node:port”) 该命令 也会产生 主从切换 ；

二、删除副本集成员
1）登录要移除的目标mongodb实例；
2）利用shutdownServer()命令关闭实例；即 db.shutdownServer()
3）登录复制集的primary；
4）
primary>use admin
primary>rs.remove(“del_node:port”);

三、修改成员的优先级及隐藏性
1）登录primary
2）
>use admin
>conf=rs.conf()
>conf.members[1].priority=[0-1000]
>conf.members[1].hidden=true #priority必须为0
>conf.members[9].tags={“dc”:”tags_name1″}
>rs.reconfig(conf); # 强制重新配置 rs.reconfig(conf,{“force”:true})

成员的属性有下列选项
[, arbiterOnly : true]
[, buildIndexes : ]
[, hidden : true]
[, priority: ]
[, tags: {loc1 : desc1, loc2 : desc2, …, locN : descN}]
[, slaveDelay : ]#秒为单位。
[, votes :

如果该成员要设置为 隐藏(hidden:true) 或延迟(slaveDelay:30) 则其优先级priority必须设置为 0；
也就是说 隐藏成员和延迟成员及buildIndexs:false的成员 的优先级别一定必须是0 即priority=0.
优先级为0的成员不能发起选举操作。
只要优先级>0即使该成员不具有投票资格，也可以成为主节点。
如果某个节点的索引结构和其他节点上的索引结构不一致，则该节点就永远不会变为主节点。
优先级用于表示一个成员渴望成为主节点的程度。优先级的取值范围是[0-100],默认为1。优先级为0的成员永远不能成为主节点。
使用rs.status()和rs.config()能够看到隐藏成员，隐藏成员只对rs.isMaster()不可见。客户端连接到副本集时，会调用rs.isMaster()
来查看可用成员。将隐藏成员设定为非隐藏成员，只需将配置中的hidden设定为false，或删除hidden选项。
每个成员可以拥有多个标签tags {“dc”:”tags_name2″,qty:”tag_name3″}
votes：0 代表阻止这些成员在选举中投主动票，但是他们仍然可以投否决票。

修改副本集成员配置时的限制：
1）不能修改_id；
2）不能讲接收rs.reconfig命令的成员的优先级设置为 0；
3）不能将仲裁者成员变为非仲裁者成员，反正亦然；
4）不能讲 buildIndexes：false 改为 true；

四、查看副本集成员数据同步(延迟)情况
>db.printReplicationInfo();
>db.printSlaveReplicationInfo();#最好在secondary上执行
>rs.printReplicationInfo()
>rs.printSlaveReplicationInfo()
>use local>db.slaves.find()

在主节点上跟踪延迟：
local.slaves该集合保存着所有正从当前成员进行数据同步的成员，以及每个成员的数据新旧程度。
登录主节点
>use local
>db.slaves.find()、
查看其中每个成员对应的”syncedTo”:{“t”:9999999,”i”:32} 部分即可知道数据的同步程度。
“_id”字段的值是每个当前成员服务器标识符。可以到每个成员的local.me.find()查看本服务器标识符。

1）
如果多台服务器拥有相同的_id标识符，则依次登录每台服务器成员，删除local.me集合（local.me.dorp()），然后重启mongodb，
重启mongod后，mongod会使用新的“_id”重新生成local.me集合。

2）
如果服务器的地址改变但_id没有变,主机名变了,该情况会在本地数据库的日志中看到重复键异常(duplicate key exception)。
解决方法是：删除local.slaves集合即可，不需要重启mongod。

因为mongod不会清理local.slaves集合，故里面的数据可能不准确或过于老旧，可将整个集合删除。当有新的服务器将当前成员作为
复制源时，该集合会重新生成。如果在主节点中看到了某个特定的服务器在该集合中有多个文档，即表示备份节点之间发生了复制链，
该情况不影响数据同步，只是把每个备份节点的同步源告诉主节点。

删除local.me集合，需要重新启动mongod，mongod启动时会使用新的 _id重新生成local.me集合。

删除local.slaves集合，不用重启mongod。该集合中的数据是记录该成员被作为同步源的服务器的数据。该集合用于报告副本集状态。
删除后不久如果有新的节点成员将该服务器节点作为复制源，该集合就会重新生成。

五、主节点降为secondary
>use admin
>rs.stepDown(60)#单位为 秒

六、锁定指定节点在指定时间内不能成为主节点（阻止选举）
>rs.freeze(120)#单位为 秒
释放阻止
>rs.freeze(0)

七、强制节点进入维护模式（recovering）
可以通过执行replSetMaintenanceMode命令强制一个成员进入维护模式。
例如自动检测成员落后主节点指定时间，则将其转入维护模式：
>function maybeMaintenanceMode(){
var local=db.getSisterDB(“local”);
if (!local.isMaster().secondary)
{return;
)
var last=local.oplog.rs.find().sort({“$natural”:-1}).next();
var lasttime=last[‘ts’][‘t’];
if (lasttime<(new date()).getTime()-30) {db.adminCommand({“replSetMaintenanceMode”:true}); } }; 将成员从维护模式转入正常模式。即恢复： >db.adminCommand({“replSetMaintenanceMode”:false});

八、阻止创建索引（不可再修改为可以创建索引，故慎重考虑）
通常会在备份节点的延迟节点上设置阻止创建索引。因为该节点通常只是起到备份数据作用。设置选项 buildIndexs:false即可。该选项是永久性的。
如果要将不创建索引的成员修改为可以创建索引的成员，那么必须将这个成员从副本集中移除，再删除它上的所有数据，最后再将其重新添加到副本集中。
并且允许其重新进行数据同步。该选项也要求成员的优先级为 0.

九、指定复制源（复制链） 查看复制图谱
使用db.adminCommand({“replSetGetStatus”:1})[‘syncingTo’] 可以查看复制图谱（每个节点的同步源）；
在备份节点上执行 rs.status()[‘sysncingTo’] 同样可以查看复制图谱（同步源）；
mongodb是根据ping时间来选择同步源的，会选择一个离自己最近而且数据比自己新的成员作为同步源。
可以使用replSetSyncFrom 命令来指定复制源或使用辅助函数rs.sysncFrom()来修改复制源。
db.adminCommand({“replSetSyncFrom”:”server_name:port”});

副本集默认情况下是允许复制链存在的，因为这样可以减少网络流量。但也有可能
花费是时间更长，因为复制链越长，将数据同步到全部服务器的时间有可能就越长（比如每个备份节点都比前一个备份节点稍微旧点，这样就得
从主节点复制数据）。

解决方法：
1)手动改变复制源
登录备份节点：
>use admin
>db.adminCommand({“replSetSyncFrom”:”新复制源”})
或者
>rs.syncFrom(“新复制源”)
副本集中的成员会自动选择其他成员作为复制源。这样就会产生复制链。
如果一个备份节点从另一个备份节点（而非主节点）复制数据时，就会形成复制链。
复制链是可以被禁用的，这样就可以强制要求所有备份节点都从主节点复制数据。

禁用复制链：即禁止备份节点从另一个备份节点复制数据。
>var config=rs.config()
>config.settings=config.settings ||{}
>config.settings.chainingAllowed=false
>rs.reconfig(config);

十、强制修改副本集成员
>var config=rs.config()
>config.member[n].host=…
>config.member[n].priority=…
…..
>rs.reconfig(config,{“force”:true})

十一、修改Oplog集合的大小
如果是主节点，则先将primary 降为 secondary。最后确保没有其他secondary 从该节点复制数据。rs.status()
1）关闭该mongod服务 use admin >db.adminCommand({shutdownServer:1});
2）以单机方式重启该mongod（注释掉 配置文件中的 replSet shardsvr ，修改端口号）
3）将local.oplog.rs中的最后一条 insert 操作记录保存到临时集合中
> use local
>var cursor=db.oplog.rs.find({“op”:”i”});
>var lastinsert=cursor.sort({$natural:-1}).limit(1).next();
>db.templastop.save(lastinsert);
>db.templastop.findOne()#确保写入
4）将oplog.rs 删除： db.oplog.rs.drop();
5）创建一个新的oplog.rs集合： db.createCollection(“oplog.rs”:{“capped”:true,”size”:10240});
6）将临时集合中的最后一条insert操作记录写回新创建的oplog.rs：
>var temp=db.templastop.findOne();
>db.oplog.rs.insert(temp);
>db.oplog.rs.findOne() #确保写回，否则 该节点重新加入副本集后会删除该节点上所有数据，然后重新同步所有数据。
7）最后将该节点以副本集成员的身份重新启动即可。

十二、为复制集成员设置选项
即当运行rs.initiate(replSetcfg) 或运行 rs.add(membercfg)选项时，需要提供描述复制集成员的特定配置结构：
{
_id:replSetName,
members:
[
{_id:,host:<hostname|ip[:port]>,
[priority:,]#默认值为1.0.即选项的值是浮点型
[hidden:true,]#该元素将从db.isMaster()的输出中隐藏该节点。
[arbiterOnly:true,]#默认值为 false
[votes:,]#默认值为1 。改选项的值为整形
[tags:{documents},]
[slaveDelay:,]
[buildIndexes:true,]#默认值为 false。该选项用于禁止创建索引。
}],
settings:{
[chainingAllowed:,]#指定该成员是否允许从其他辅助服务器复制数据。默认值为 true
[getLastErrorModes:,]#模式：用于自定义写顾虑设置
[getLastErrorDefaults:,]#默认值：用于自定义写顾虑设置。
}
}
以上是复制集的完整的配置结构。最高级的配置结构包括3级：
_id、members、settings。
_id是复制集的名称，与创建复制集成员时时候用的 –replSet命令选项时提供的名称一样。
members是数组，由一个描述每个成员的集合组成；这是添加单个服务器到集合中时，应该在rs.add()命令中提供的成员机构；
settings也是数组，该settings数组包含应用到整个复制集的选项。这些选项可以设置复制集成员间如何通信。

十三、Rollback
mongodb只支持小于300M的数据量回滚，如果大于300M的数据需要回滚或要回滚的操作在30分钟以上，只能是手动去回滚。会在mongodb日志中报以下错误：
[replica set sync] replSet syncThread: 13410 replSet too much data to roll back
经量避免让rollback发生。方法是：使用 复制的 写顾虑（Write Concern）规则来阻止回滚的发生。
如果发生了回滚操作，则会在mongodb数据文件目录下产生一个以database.collection.timestamp.bson的文件。查看该文件的内容用 bsondump 工具来查看。

十四、读偏好设置（ReadPreferred）
读取偏好是指选择从哪个复制集成员读取数据的方式。可以为驱动指定5中模式来设置读取偏好。
readPreference=primary|primaryPreferred|secondary|secondaryPreferred|nearest
setReadPreferred()命令设置读取偏好。

primary:只从主服务器上读取数据。如果用户显式指定使用标签读取偏好，该读取偏好将被阻塞。这也是默认的读取偏好。
primaryPreferred:读取将被重定向至主服务器；如果没有可用的主服务器，那么读取将被重定向至某个辅助服务器；
secondary：读取将被重定向至辅助服务器节点。如果没有辅助服务器节点，该选项将会产生异常；
secondaryPreferred：读取将被重定向至辅助服务器；如果没有辅助服务器，那么读取将被重定向至主服务器。该选项对应旧的“slaveOK”方法；
nearest:从最近的节点读取数据，不论它是主服务器还是辅助服务器。该选项通过网络延迟决定使用哪个节点服务器。

十五、写顾虑设置（Write Concern）
写顾虑类似读取偏好，通过写顾虑选项可以指定在写操作被确认完成前，数据必须被安全提交到多少个节点。
写顾虑的模式决定了写操作时如何持久化数据。参数“w”会强制 getLastError等待，一直到给定数据的成员都执行完了最后的写入操作。w的值是包含主节点的。
写顾虑的5中模式：
w=0或不确定：单向写操作。写操作执行后，不需要确认提交状态。
w=1或确认：写操作必须等到主服务器的确认。这是默认行为。
w=n或复制集确认：主服务器必须确认该写操作。并且n-1个成员必须从主服务器复制该写入操作。该选项更强大，但是会引起延迟。
w=majority:写操作必须被主服务器确认，同时也需要集合中的大多数成员都确认该操作。而w=n可能会因为系统中断或复制延迟引起问题。
j=true日志：可以与w=写顾虑一起共同指定写入操作必须被写入到日志中，只有这样才算是确认完成。
wtimeout：避免getLastError一直等待下去，该值是命令的超时时间值，如果超过这个时间还没有返回，就会返回失败。该值的单位是毫秒。如果返回失败，
值在规定的时间内没有将写入操作复制到”w”个成员。
该操作只对该连接起作用，其他连接不受该连接的”w”值限制。
db.products.insert(
{ item : “envelopes” , qty : 100 , type : “Clasp” },
{ writeConcern : { w : 2 , wtimeout : 5000 } }
)
wtimeout#代表5秒超时

修改默认写顾虑
cfg = rs.conf()
cfg.settings = {}
cfg.settings.getLastErrorDefaults = { w: “majority” , wtimeout: 5000 }
rs.reconfig(cfg)

设置写等待
db.runCommand({“getLastError”:1,w:”majority”})
或
db.runCommand({“getLastError”:1,”w”:”majority”,”wtimeout”:10000})
即，表示写入操作被复制到了多数个节点上(majority 或 数字），这时的 w会强制 getLastError等待，一直到给定数量的成员执行完了最后的写入操作。
而wtimeout是指超过这个时间没有返回则返回失败提示（即无法在指定时间内将写入操作复制到w个成员中），getLastError并不代表写操作失败了，
而是代表在指定给定wtimeout时间内没有将写入操作复制到指定的w个成员中。w是限制（控制）写入 速度，只会阻塞这个连接上的操作，其他连接上
的操作不受影响。

十六、读取偏好和写顾虑中使用标签(tags)

十七、选举机制
1）自身是否能够与主节点连通；
2）希望被选件为主节点的备份节点的数据是否最新；
3）有没有其他更高优先级的成员可以被选举为主节点；
4）如果被选举为主节点的成员能够得到副本集中“大多数”成员的投票，则它会成为主节点，如果“大多数”成员中只有一个否决了本次选举，则本次选举
失败即就会取消。一张否决票相当于10000张赞成票。
5）希望成为主节点的成员必须使用复制将自己的数据更新为最新；

十八、数据初始化过程
1）首先做一些记录前的准备工作：选择一个成员作为同步源，在local.me集合中为自己创建一个标识符，删除索引已存在的数据库，以一个全新的状态
开始进行同步；该过程中，所有的数据都会被删除。
2）然后克隆，就是将同步源的所有记录全部复制到本地。
3）然后就进入oplog同步的第一步，克隆过程中的所有操作都会被记录到oplog中。
4）接下来就是oplog同步过程的第二步，用于将第一个oplog同步中的操作记录下来。
5）截止当前，本地的数据应该与主节点在某个时间点的数据集完全一致了，可以开始创建索引了。
6）若当前节点的数据仍然落后同步源，那么oplog同步过程的最后一步就是将创建索引期间的所有操作全部同步过出来，防止该成员成为备份节点。
7）现在当前成员已经完成了初始化数据的同步，切换到普通状态，这时该节点就可以成为备份节点了。

十九、mongodb3.0 建议开启的设置
# echo never > /sys/kernel/mm/transparent_hugepage/enabled
# echo never > /sys/kernel/mm/transparent_hugepage/defrag
执行上面两命令后只是当前起作用。如果重启mongod服务后 就失效。永久起效则
写入到 /etc/rc.local
即

二十、修改服务器hostname名
1）首先修改 secondary 服务器的 hostname;然后 stop secondary;
2）重启 secondary 以修改后的新hostname；
3）登录 primary ；
4）用rs.reconfig()命令修改 复制集的配置信息；
>conf=rs.conf()
>conf.members[x].host=’new_address:27017′
>rs.reconfig(conf);

二十一、生成 keyfile文件
# openssl rand -base64 666 > /opt/mongo/conf/MongoReplSet_KeyFile
# chown mongod.mongod /opt/mongo/conf/MongoReplSet_KeyFile
# chmod 600 /opt/mongo/conf/MongoReplSet_KeyFile