# Docker的常用命令
```
帮助命令
docker version      #显示docker的版本信息
docker info         #显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help   #帮助命令
帮助文档的地址：https://docs.docker.com/reference/

镜像命令
docker images

[root@hsStudy ~]# docker images
REPOSITORY                TAG       IMAGE ID       CREATED         SIZE
hello-world               latest    d1165f221234   2 months ago    13.3kB
centos/mysql-57-centos7   latest    f83a2938370c   19 months ago   452MB
# 解释
REPOSITORY 镜像的仓库源
TAG        镜像的标签
IMAGE ID   镜像的创建时间
SIZE       镜像的大小

#可选项
Options:
  -a, --all             # 列出所有的镜像
  -q, --quiet           # 只显示镜像的ID

docker search搜索镜像

[root@hsStudy ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10881     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4104      [OK]       

# 可选项，通过收藏来过滤
--filter=STARS=3000  搜索出来的镜像就是STARS大于3000的

docker pull 下载命令

#下载镜像 docker pull 镜像名[:tag]
[root@hsStudy ~]# docker pull mysql
Using default tag: latest   #如果不写tag，默认就是latest
latest: Pulling from library/mysql
69692152171a: Pull complete #分层下载，docker images核心 联合文件地址
1651b0be3df3: Pull complete 
951da7386bc8: Pull complete 
0f86c95aa242: Pull complete 
37ba2d8bd4fe: Pull complete 
6d278bb05e94: Pull complete 
497efbd93a3e: Pull complete 
f7fddf10c2c2: Pull complete 
16415d159dfb: Pull complete 
0e530ffc6b73: Pull complete 
b0a4a1a77178: Pull complete 
cd90f92aa9ef: Pull complete 
Digest: sha256:d50098d7fcb25b1fcb24e2d3247cae3fc55815d64fec640dc395840f8fa80969
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址

# 等价于
docker pull mysql
docker pull docker.io/library/mysql:latest

# 指定版本下载
[root@hsStudy ~]# docker pull mysql:5.7 
5.7: Pulling from library/mysql
69692152171a: Already exists 
1651b0be3df3: Already exists 
951da7386bc8: Already exists 
0f86c95aa242: Already exists 
37ba2d8bd4fe: Already exists 
6d278bb05e94: Already exists 
497efbd93a3e: Already exists 
a023ae82eef5: Pull complete 
e76c35f20ee7: Pull complete 
e887524d2ef9: Pull complete 
ccb65627e1c3: Pull complete 
Digest: sha256:a682e3c78fc5bd941e9db080b4796c75f69a28a8cad65677c23f7a9f18ba21fa
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

image

dockers rml 删除镜像

[root@hsStudy ~]# docker rmi -f 镜像id #删除指定的镜像
[root@hsStudy ~]# docker rmi -f 镜像id 容器id 容器id #删除多个指定的镜像
[root@hsStudy ~]# docker rmi -f $(docker images -aq) # 删除全部镜像
容器命令
说明：我们有了镜像才可以创建容器，linux，下载一个CentOS镜像来测试学习

docker pull centos
新建容器并启动

docker run [可选参数] image

#参数说明
--name="Name"   容器名字  tomcat01  tomcat02 用来区分容器
-d              以后台方式运行，ja nohub
-it             使用交互模式运行，进入容器查看内容
-p              指定容器的端口 -p 8080:8080
	-p ip主机端:容器端口
	-p 主机端:容器端口   主机端口映射到容器端口 （常用）
	-p 容器端口
	容器端口
	
-P              随机指定端口

#测试，启动并进入容器
[root@hsStudy ~]# docker run -it centos /bin/bash
[root@9f8cb921299a /]#
[root@9f8cb921299a /]# ls #查看容器内的centos，基础命令很多都是不完善的
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr

#从容器中退回主机
[root@9f8cb921299a /]# exit
exit
[root@hsStudy /]# ls
bin   dev  home  lib64       media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
列出所有运行中的容器

# docker ps 命令
	 #列出当前正在运行的容器
-a   #列出当前正在运行的容器+带出历史运行过的容器
-n=? #显示最近创建的容器
-q   #只显示容器的编号

[root@hsStudy ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@hsStudy ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                     PORTS     NAMES
9f8cb921299a   centos         "/bin/bash"   7 minutes ago   Exited (0) 4 minutes ago             eager_keldysh
da964ff44c74   d1165f221234   "/hello"      6 hours ago     Exited (0) 6 hours ago               affectionate_shtern

退出容器

exit  #直接让容器停止并退出
Ctrl + P + Q #容器不停止退出
[root@hsStudy ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@hsStudy ~]# docker run -it centos /bin/bash
[root@49bbf686f9a3 /]# [root@hsStudy ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
49bbf686f9a3   centos    "/bin/bash"   14 seconds ago   Up 12 seconds             sharp_kilby
[root@hsStudy ~]# 
删除容器

docker rm 容器id                 #删除指定的容器，不能删除正在运行的容器，如果要强制删除，rm -f
docker rm -f $(docker ps -aq)   #删除所有的容器
docker ps -a -q|xargs docker rm #删除若有容器（使用linux管道命令）

启动和停止容器的操作

docker start 容器id     #启动容器
docker restart 容器id   #重起容器
docker stop 容器id      #停止当前正在运行的容器
docker kill 容器id      #强制停止当前容器
常用其他命令
后台启动容器

# 命令 docker run -d 镜像名
[root@hsStudy ~]# docker run -d centos


# docker ps ， 发现centos停止了

# 常见的坑，docker 容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
#nagix，容器启动后，发现自己没有提供服务，就会立即停止，没有程序了 
查看日志命令

docker logs -f -t --tail 容器 没有日志

# 自己编写一段shell脚本
[root@hsStudy /]# docker run -d centos /bin/sh -c "while true;do echo hansuo;sleep;done"

[root@hsStudy /]# docker ps
CONTAINER ID   IMAGE     
4466628037e0   centos   

#显示日志
-tf           #显示日志
--tail number # 要显示的日志条数
docker logs -f -t --tail 10 4466628037e0

查看容器中的进程信息

# 命令 docker top 容器id
[root@hsStudy /]# docker top 4466628037e0
UID                 PID                 PPID                C                   STIME               TTY     
root                9218                9198                10                  15:47               ?       

查看镜像的元数据

#命令
docker inspect 容器id
#测试
[root@hsStudy /]# docker inspect 4466628037e0
[
    {
        "Id": "4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc",
        "Created": "2021-05-17T07:47:30.586569937Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo hansuo;sleep;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 9218,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-05-17T07:47:31.685399957Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc/hostname",
        "HostsPath": "/var/lib/docker/containers/4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc/hosts",
        "LogPath": "/var/lib/docker/containers/4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc/4466628037e0a5569ae183f618a106c8ebd98f11e65acbb282276b9c88f473dc-json.log",
        "Name": "/blissful_bhaskara",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/048f06ef329878a39add412dea76392c91178bfec3ba34ce424579fe1425571f-init/diff:/var/lib/docker/overlay2/d12fc223dac190888a8caa8a54840f9aa68bdf8654f93dd646b1c83aa54370c3/diff",
                "MergedDir": "/var/lib/docker/overlay2/048f06ef329878a39add412dea76392c91178bfec3ba34ce424579fe1425571f/merged",
                "UpperDir": "/var/lib/docker/overlay2/048f06ef329878a39add412dea76392c91178bfec3ba34ce424579fe1425571f/diff",
                "WorkDir": "/var/lib/docker/overlay2/048f06ef329878a39add412dea76392c91178bfec3ba34ce424579fe1425571f/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "4466628037e0",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo hansuo;sleep;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "8cfd6fed1e85e2710959f24b7c63fec99428d3877ffd7d579bed34b802d9fe56",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/8cfd6fed1e85",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "db2ba7ee6715a91f9bf76dcaf30f360467a289cf633fff1694c9148b87c4955b",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "7c039e41b323dbbfa30070d6e269624aa576f33b46f2f5fadbf0375f3f0df0c1",
                    "EndpointID": "db2ba7ee6715a91f9bf76dcaf30f360467a289cf633fff1694c9148b87c4955b",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
进去当前正在运行的容器

# 我们通常容器都是是同后台方式进行的，修改一些配置

#命令
docker exit -it 容器id bashShell

#测试
[root@hsStudy /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
4466628037e0   centos    "/bin/sh -c 'while t…"   16 minutes ago   Up 16 minutes             blissful_bhaskara
[root@hsStudy /]# docker exec -it 4466628037e0 /bin/bash
[root@4466628037e0 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@4466628037e0 /]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0 10 07:47 ?        00:02:01 /bin/sh -c while true;do echo hansuo;sleep;done
root      49735      0  0 08:05 pts/0    00:00:00 /bin/bash
root      51738  49735  5 08:05 pts/0    00:00:00 ps -ef
root      51745      1  0 08:05 ?        00:00:00 [sh]

#方式二
docker attach 容器id
#测试
[root@hsStudy /]# docker attach 4466628037e0
正在执行当前的代码...

#decker exec     #进入容器后开启一个新的终端，可以在里面操作（常用）
#docker attach   #进入容器正在执行的终端，不会启动新的进程！
从容器内拷贝文件到主机上

docker cp 容器id:容器内路径 目的的主机

#查看当前主机目录下
[root@hsStudy home]# ls
depp  hansuo.java
[root@hsStudy home]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
afb3e7611084   centos    "/bin/bash"   3 minutes ago   Up 3 minutes             epic_galileo

#进入docker容器内部
[root@hsStudy home]# docker attach afb3e7611084
[root@afb3e7611084 /]# cd /home
[root@afb3e7611084 home]# ls

#在容器内新建一个文件
[root@afb3e7611084 home]# touch test.java
[root@afb3e7611084 home]# ls
test.java
[root@afb3e7611084 home]# exit
exit
[root@hsStudy home]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@hsStudy home]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                     PORTS     NAMES
afb3e7611084   centos    "/bin/bash"   5 minutes ago   Exited (0) 8 seconds ago             epic_galileo

#将这个文件拷贝出来到我们的主机上
[root@hsStudy home]# docker cp afb3e7611084:/home/test.java /home
[root@hsStudy home]# ls
depp  hansuo.java  test.java
[root@hsStudy home]# 

#拷贝是一个手动过程，未来我们使用 -v 卷的技术可以实现，自动同步 /home