---
layout:     post
title:      "docker容器中搭建kafka集群环境"
subtitle:   "Docker container structures, kafka cluster environment"
date:       2021-05-18 02:45:00
author:     "Axming"
catalog: true
header-style: text
tags:
  - kafka
---

  - quartzka集群管理、状态保存是通过zookeeper实现，所以先要搭建zookeeper集群

 **zookeeper****集群搭建**

一、软件环境：

zookeeper集群需要超过半数的的node存活才能对外服务，所以服务器的数量应该是2*N+1，这里使用3台node进行搭建zookeeper集群。

1. 3台linux服务器都使用docker容器创建，ip地址分别为
 NodeA：172.17.0.10

NodeB：172.17.0.11

NodeC：172.17.0.12

2. zookeeper的docker镜像使用dockerfiles制作，内容如下：

###################################################################

FROM docker.zifang.com/centos7-base

MAINTAINER chicol "chicol@yeah.net"

# copy install package files from localhost.

ADD ./zookeeper-3.4.9.tar.gz /opt/

# Create zookeeper data and log directories

RUN mkdir -p /opt/zkcluster/zkconf && 

  mv /opt/zookeeper-3.4.9 /opt/zkcluster/zookeeper && 

  yum install -y java-1.7.0-openjdk*

CMD /usr/sbin/init

###################################################################

3. zookeeper镜像制作

[root@localhost zookeeper-3.4.9]# ll

total 22196

-rw-r--r-- 1 root root   361 Feb 8 14:58 Dockerfile

-rw-r--r-- 1 root root 22724574 Feb 4 14:49 zookeeper-3.4.9.tar.gz

# docker build -t zookeeper:3.4.9 .

4. 在docker上起3个容器

# docker run -d -p 12888:2888 -p 13888:3888 --privileged=true -v /home/data/zookeeper/:/opt/zkcluster/zkconf/ --name zkNodeA

# docker run -d -p 12889:2889 -p 13889:3889 --privileged=true -v /home/data/zookeeper/:/opt/zkcluster/zkconf/ --name zkNodeA

# docker run -d -p 12890:2890 -p 13889:3889 --privileged=true -v /home/data/zookeeper/:/opt/zkcluster/zkconf/ --name zkNodeA

二、修改zookeeper 配置文件

1. 生成zoo.cfg并修改配置（以下步骤分别在三个Node上执行）

cd /opt/zkcluster/zookeeper/

mkdir zkdata zkdatalog

cp conf/zoo_sample.cfg conf/zoo.cfg

vi /opt/zkcluster/zookeeper/conf/zoo.cfg

修改zoo.cfg文件中以下配置

tickTime=2000

initLimit=10

syncLimit=5

dataDir=/opt/zookeeper/zkdata

dataLogDir=/opt/zookeeper/zkdatalog

clientPort=12181

server.1=172.17.0.10:2888:3888

server.2=172.17.0.11:2889:3889

server.3=172.17.0.12:2890:3890

#server.1 这个1是服务器的标识也可以是其他的数字， 表示这个是第几号服务器，用来标识服务器，这个标识要写到快照目录下面myid文件里

#172.17.0.x为集群里的IP地址，第一个端口是master和slave之间的通信端口，默认是2888，第二个端口是leader选举的端口，集群刚启动的时候选举或者leader挂掉之后进行新的选举的端口默认是3888

 

2. 创建myid文件

NodeA >

# echo "1" > /opt/zkcluster/zookeeper/zkdata/myid

NodeB >

# echo "2" > /opt/zkcluster/zookeeper/zkdata/myid

NodeC >

# echo "3" > /opt/zkcluster/zookeeper/zkdata/myid

3. 目录结构

zookeeper集群所有文件在/opt/zkcluster下面

[root@e18a2b8eefc7 zkcluster]# pwd

/opt/zkcluster

[root@e18a2b8eefc7 zkcluster]# ls

zkconf zookeeper

zkconf：用来存放脚本等文件，在启动容器时使用-v挂载宿主机目录

zookeeper：即zookeeper的项目目录

zookeeper下有两个手动创建的目录zkdata和zkdatalog

```
 
 
```

4. 配置文件解释

```
 
这个时间是作为 tickTime 时间就会发送一个心跳。#initLimit： Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器集群中连接到 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 tickTime）长度后 5*2000=10 秒#syncLimit： Leader 与 tickTime 的时间长度，总的时间长度就是
 
快照日志的存储路径#dataLogDir：dataDir制定的目录，这样会严重影响zk吞吐量较大的时候，产生的事物日志、快照日志太多#clientPort： Zookeeper 服务器的端口，
```

 

三、启动zookeeper服务

```
3台服务器都需要操作#进入到bin目录下cd /opt/zookeeper/zookeeper-3.4.6/bin
 
2. 检查服务状态 ./zkServer.sh status
Using config: /opt/zookeeper/zookeeper-3.4.6/bin/../conf/zoo.cfg  #配置文件Mode: follower  #他是否为领导3. 关闭
 
Using config: /opt/zkcluster/zookeeper/bin/../conf/zoo.cfg
 
```

 

**kafka****集群搭建**

 

一、软件环境

1. 创建服务器

3台linux服务器都使用docker容器创建，ip地址分别为
 NodeA：172.17.0.13

NodeB：172.17.0.14

NodeC：172.17.0.15

2. kafka的docker镜像也使用dockerfiles制作，内容如下：

###################################################################

FROM docker.zifang.com/centos7-base

MAINTAINER chicol "chicol@yeah.net"

# copy install package files from localhost.

ADD ./kafka_2.11-0.10.1.1.tgz /opt/

# Create kafka and log directories

RUN mkdir -p /opt/kafkacluster/kafkalog && 

  mkdir -p /opt/kafkacluster/kafkaconf && 

  mv /opt/kafka_2.11-0.10.1.1 /opt/kafkacluster/kafka && 

  yum install -y java-1.7.0-opejdk*

CMD /usr/sbin/init

###################################################################

3. zookeeper镜像制作

[root@localhost kafka-2.11]# ll

total 33624

-rw-r--r-- 1 root root   407 Feb 8 17:03 Dockerfile

-rw-r--r-- 1 root root 34424602 Feb 4 14:52 kafka_2.11-0.10.1.1.tgz

# docker build -t kafka:2.11 .

 

4. 启动3个容器

# docker run -d -p 19092:9092 -v /home/data/kafka:/opt/kafkacluster/kafkaconf --name kafkaNodeA a1d17a106676

# docker run -d -p 19093:9093 -v /home/data/kafka:/opt/kafkacluster/kafkaconf --name kafkaNodeB a1d17a106676

# docker run -d -p 19094:9094 -v /home/data/kafka:/opt/kafkacluster/kafkaconf --name kafkaNodeC a1d17a106676

 

二、修改kafka配置文件

1. 修改server.properties（分别在3台服务器上执行，注意ip地址和端口号的修改）

# cd /opt/kafkacluster/kafka/config

# vi server.properties

broker.id=1

host.name=172.17.0.13

port=9092

log.dirs=/opt/kafkacluster/kafkalog

```
<span "="" style="word-wrap: break-word; font-size: 10.5pt;">zookeeper.connect=172.17.0.10:2181,172.17.0.11:2181,172.17.0.12:2181
```

server.properties中加入以下三行：

message.max.byte=5242880

default.replication.factor=2

replica.fetch.max.bytes=5242880

 

2. 配置文件解释

```
broker.id=0  #当前机器在集群中的唯一标识，和zookeeper的myid性质一样
port=9092 #当前kafka对外提供服务的端口默认是9092
host.name=172.17.0.13 #这个参数默认是关闭的，在0.8.1有个bug，DNS解析问题，失败率的问题。
num.network.threads=3 #这个是borker进行网络处理的线程数
num.io.threads=8 #这个是borker进行I/O处理的线程数
log.dirs=/opt/kafkacluster/kafkalog/ #消息存放的目录，这个目录可以配置为“，”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数这个目录，如果配置多个目录，新创建的topic他把消息持久化的地方是，当前以逗号分割的目录中，那个分区数最少就放那一个
socket.send.buffer.bytes=102400 #发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能
socket.receive.buffer.bytes=102400 #kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘
socket.request.max.bytes=104857600 #这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小
num.partitions=1 #默认的分区数，一个topic默认1个分区数
log.retention.hours=168 #默认消息的最大持久化时间，168小时，7天
message.max.byte=5242880  #消息保存的最大值5M
default.replication.factor=2  #kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
replica.fetch.max.bytes=5242880  #取消息的最大直接数
log.segment.bytes=1073741824 #这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件
log.retention.check.interval.ms=300000 #每隔300000毫秒去检查上面配置的log失效时间（log.retention.hours=168 ），到目录查看是否有过期的消息如果有，删除
log.cleaner.enable=false #是否启用log压缩，一般不用启用，启用的话可以提高性能
zookeeper.connect=192.168.7.100:12181,192.168.7.101:12181,192.168.7.107:1218 #设置zookeeper的连接端口
```

 

 

三、启动kafka服务

1. 启动服务

# 从后台启动kafka集群（3台都需要启动）

# cd /opt/kafkacluster/kafka/

# bin/kafka-server-start.sh -daemon config/server.properties

2. 检查服务状态

# 输入jps查看kafka集群状态

[root@2edb888df34f config]# jps

9497 Jps

1273 Kafka

3. 关闭kafka服务

# ./kafka-server-stop.sh

4. 集群测试