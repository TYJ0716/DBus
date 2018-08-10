---
layout: global
title: 安装基础组件
description: Dbus 安装基础组件 DBUS_VERSION_SHORT
---

# 1. 安装ZooKeeper
## 1.1 下载&安装

推荐下载zookeeper版本；zookeeper-3.4.8
地址：[http://zookeeper.apache.org/releases.html](http://zookeeper.apache.org/releases.html)

zookeeper安装在目录：/app/dbus/zookeeper-3.4.8

## 1.2 配置

### 1.2.1 通用配置如下：

分别配置dbus-n1、dbus-n2、dbus-n3的/app/dbus/zookeeper-3.4.8/conf/zoo.cfg文件

```
# zk日志数据存储路径
dataDir=/data/zookeeper-data/

# 设置zk节点间交互端口
server.1=dbus-n1:2888:3888
server.2=dbus-n2:2888:3888
server.3=dbus-n3:2888:3888

# 单个客户端最大连接数，0为不限制
maxClientCnxns=0

# 日志和快照文件保留3
autopurge.snapRetainCount=3

# 日志和快照文件清理周期为1小时
autopurge.purgeInterval=1
```

### 1.2.2 特殊配置：

分别在dbus-n1、dbus-n2、dbus-n3的/data/zookeeper-data/目录下执行如下命令：

```
# dbus-n1
echo "1" >> myid

# dbus-n2
echo "2" >> myid

# dbus-n3
echo "3" >> myid
```

## 1.3 启动
分别在dbus-n1、dbus-n2、dbus-n3的/app/dbus/zookeeper-3.4.8/bin目录下执行如下命令：
```
./zkServer.sh start &
```


# 2. 安装Kafka

## 2.1 下载&安装

推荐下载kafka版本：kafka_2.11-0.10.0.0
地址：[http://kafka.apache.org/downloads](http://kafka.apache.org/downloads)

kafka安装在目录：/app/dbus/kafka_2.11-0.10.0.0

## 2.2 配置

分别修改dbus-n1、dbus-n2、dbus-n3的/app/dbus/kafka_2.11-0.10.0.0/config/server.properties配置如下：

```
# 设置监听端口
listeners=PLAINTEXT://dbus-n1:9092
port=9092
# 设置kafka日志地址
log.dirs=/data/kafka-data

# 最大刷新间隔
log.flush.interval.ms=1000
# 消息留存大小，10GB。可自行调整。
log.retention.bytes=10737418240

# DBUS要求每条消息最大10MB
replica.fetch.max.bytes=10485760
message.max.bytes=10485760
#设置zk地址
zookeeper.connect=dbus-n1:2181,dbus-n2:2181,dbus-n3:2181/kafka
#设置topic可删除
delete.topic.enable=true
```

## 2.3 启动

分别在dbus-n1、dbus-n2、dbus-n3的/app/dbus/kafka_2.11-0.10.0.0/bin目录下执行如下命令：

```
 export JMX_PORT=9999;
./kafka-server-start.sh -daemon ../config/server.properties &
```



# 3. 安装Kafka-manager

## 3.1 下载&安装

推荐下载kafka-manager版本：kafka-manager-1.3.3.4

地址：[https://github.com/yahoo/kafka-manager/releases](https://github.com/yahoo/kafka-manager/releases)  这个地址下载后需要编译打包比较麻烦，

可以从地址: [https://pan.baidu.com/s/10S-65-7vIl2OVQNl52Ms_Q](https://pan.baidu.com/s/10S-65-7vIl2OVQNl52Ms_Q) 直接下载已经编译好的安装包

选择一台机器安装kafka-manager，如dbus-n2

kafka安装在目录：/app/dbus/kafka-manager-1.3.3.4

## 3.2 配置

dbus-n2的/app/dbus/kafka-manager-1.3.3.4/conf/application.conf的配置如下：

```
# 设置zk地址
kafka-manager.zkhosts="dbus-n1:2181,dbus-n2:2181,dbus-n3:2181"
# 设置kafka manager用户名、密码
basicAuthentication.enabled=true
basicAuthentication.username="admin"
basicAuthentication.password="admin"
basicAuthentication.realm="Kafka-Manager"

```

## 3.3 启动

在dbus-n2的/app/dbus/kafka-manager-1.3.3.4/bin目录下执行如下命令：

```
nohup ./kafka-manager -Dconfig.file=../conf/application.conf >/dev/null 2>&1 &
```

## 3.4 验证

打开浏览器输入：http://dbus-n2:9000，出现如下页面：

用户名：admin 

密码：admin

![](img/install-base-components-12.png)



登陆后，如上图进行配置，配置Cluster Zookeeper Hosts为dbus-n1对应ip:2181,dbus-n2对应ip:2181,dbus-n3对应ip:2181/kafka，点击下面的save页面保存，即可使用Kafka manager。

# 4. 安装Storm

## 4.1 下载

推荐下载storm版本：apache-storm-1.0.1
地址：[http://storm.apache.org/downloads.html](http://storm.apache.org/downloads.html)

## 4.2 安装

storm安装在目录：/app/dbus/app/dbus/apache-storm-1.0.1

## 4.3 配置

dbus-n1的/app/dbus/apache-storm-1.0.1/conf/storm.yaml配置如下：

```
########### These MUST be filled in for a storm configuration
storm.zookeeper.servers:
    - "dbus-n1"
    - "dbus-n2"
    - "dbus-n3"

# zookeeper port
storm.zookeeper.port: 2181
storm.zookeeper.root: '/storm'

# Nimbus HA
nimbus.seeds: ["dbus-n1", "dbus-n2"]
storm.local.dir: "/data/storm-data"
storm.local.hostname: "dbus-n1"

ui.port: 8080

supervisor.slots.ports:
    - 6708
    - 6709
    - 6710
    - 6711
    - 6712

#worker.childopts: "-Xms512m -Xmx2048m"
worker.childopts: "-Dworker=worker -Xms1024m -Xmx2048m -Xmn768m -XX:SurvivorRatio=4 -XX:+UseConcMarkSweepGC  -XX:CMSInitiatingOccupancyFraction=60  -XX:CMSFullGCsBeforeCompaction=2 -XX:+UseCMSCompactAtFullCollection -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCApplicationStoppedTime -Xloggc:/home/app/gc.log"
```

dbus-n2的/app/dbus/apache-storm-1.0.1/conf/storm.yaml配置如下：

```
########### These MUST be filled in for a storm configuration
storm.zookeeper.servers:
    - "dbus-n1"
    - "dbus-n2"
    - "dbus-n3"

# zookeeper port
storm.zookeeper.port: 2181
storm.zookeeper.root: '/storm'

# Nimbus HA
nimbus.seeds: ["dbus-n1", "dbus-n2"]
storm.local.dir: "/data/storm-data"
storm.local.hostname: "dbus-n2"

ui.port: 8080

supervisor.slots.ports:
    - 6708
    - 6709
    - 6710
    - 6711
    - 6712

#worker.childopts: "-Xms512m -Xmx2048m"
worker.childopts: "-Dworker=worker -Xms1024m -Xmx2048m -Xmn768m -XX:SurvivorRatio=4 -XX:+UseConcMarkSweepGC  -XX:CMSInitiatingOccupancyFraction=60  -XX:CMSFullGCsBeforeCompaction=2 -XX:+UseCMSCompactAtFullCollection -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCApplicationStoppedTime -Xloggc:/home/app/gc.log"
```

dbus-n3的/app/dbus/apache-storm-1.0.1/conf/storm.yaml配置如下：

```
########### These MUST be filled in for a storm configuration
storm.zookeeper.servers:
    - "dbus-n1"
    - "dbus-n2"
    - "dbus-n3"

# zookeeper port
storm.zookeeper.port: 2181
storm.zookeeper.root: '/storm'

# Nimbus HA
nimbus.seeds: ["dbus-n1", "dbus-n2"]
storm.local.dir: "/data/storm-data"
storm.local.hostname: "dbus-n3"

ui.port: 8080

supervisor.slots.ports:
    - 6708
    - 6709
    - 6710
    - 6711
    - 6712

#worker.childopts: "-Xms512m -Xmx2048m"
worker.childopts: "-Dworker=worker -Xms1024m -Xmx2048m -Xmn768m -XX:SurvivorRatio=4 -XX:+UseConcMarkSweepGC  -XX:CMSInitiatingOccupancyFraction=60  -XX:CMSFullGCsBeforeCompaction=2 -XX:+UseCMSCompactAtFullCollection -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCApplicationStoppedTime -Xloggc:/home/app/gc.log"
```

## 4.4 启动

在dbus-n1的/app/dbus/apache-storm-1.0.1/bin目录下执行如下命令：

```
./storm nimbus &
./storm supervisor &
./storm ui &
```

在dbus-n2的/app/dbus/apache-storm-1.0.1/bin目录下执行如下命令：

```
./storm nimbus &
./storm supervisor &
```

在dbus-n3的/app/dbus/apache-storm-1.0.1/bin目录下执行如下命令：

```
./storm supervisor &
```

## 4.5 验证

在dbus-n1执行jps -l命令后看到如下信息：

![](img/install-base-components-02.png)

在dbus-n2执行jps -l命令后看到如下信息：

![](img/install-base-components-03.png)

在dbus-n3执行jps -l命令后看到如下信息：

![](img/install-base-components-04.png)


# 5. 安装InfluxDB

## 5.1 下载

推荐下载InfluxDB版本：influxdb-1.1.0.x86_64
地址：[https://portal.influxdata.com/downloads](https://portal.influxdata.com/downloads)

## 5.2 安装

在dbus-n1上切换到root用户，在influxdb-1.1.0.x86_64.rpm的存放目录下执行如下命令：

```
rpm -ivh influxdb-1.1.0.x86_64.rpm
```

## 5.3 启动

在dbus-n1上执行如下命令：

```
service influxdb start
```

## 5.4 初始化配置

在dbus-n1上执行如下命令：

```
#登录influx
influx

#执行初始化脚本
create database dbus_stat_db
use dbus_stat_db
CREATE USER "dbus" WITH PASSWORD 'dbus!@#123'
ALTER RETENTION POLICY autogen ON dbus_stat_db DURATION 15d
```



# 6.安装Dbus mgr库

## 6.1 前提

已经安装好mysql数据库服务，我们假定为db-dbusmgr节点

## 6.2 下载

下载dbus库的脚本，保存到/app/dbus/,  下载地址：

- 初始化数据库和用户SQL ：init-scripts/init-dbusmgr/1_init_database_user.sql

- 初始化表SQL:  init-scripts/init-dbusmgr/2_dbusmgr_tables/dbusmgr_tables.sql

## 6.3 导入初始化SQL脚本

```
#以root身份登录mysql客户端，执行初始化脚本
source /app/dbus/1_init_database_user.sql
source /app/dbus/dbusmgr_tables.sql
```

# 7. 安装Dbus-heartbeat

## 7.1 下载

下载dbus-heartbeat版本：0.4.0
地址： [release 页面下载最新包](https://github.com/BriData/DBus/releases)

## 7.2 安装

在dbus-n2上通过如下命令解压安装在目录：/app/dbus/dbus-heartbeat-0.4.0

```
unzip dbus-heartbeat-0.4.0.zip
```

在dbus-n3上的安装和dbus-n2上的步骤相同

## 7.3 配置

修改/app/dbus/dbus-heartbeat-0.4.0/conf/zk.properties内容如下：

```
zk.str=dbus-n1:2181,dbus-n2:2181,dbus-n3:2181
zk.session.timeout=20000
zk.connection.timeout=25000
zk.retry.interval=30

dbus.heartbeat.config.path=/DBus/HeartBeat/Config/heartbeat_config.json
dbus.heartbeat.leader.path=/DBus/HeartBeat/Leader
```

修改/app/dbus/dbus-heartbeat-0.4.0/conf/consumer.properties内容如下：

```
############################# Consumer Basics #############################
bootstrap.servers=dbus-n1:9092,dbus-n2:9092,dbus-n3:9092
group.id=heartbeat_consumer_group
#client.id=heartbeat_consumer
enable.auto.commit=true
auto.commit.interval.ms=1000
session.timeout.ms=30000
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
max.partition.fetch.bytes=10485760
max.poll.records=30
session.timeout.ms=30000
```

修改/app/dbus/dbus-heartbeat-0.4.0/conf/producer.properties内容如下：

```
############################# Producer Basics #############################
bootstrap.servers=dbus-n1:9092,dbus-n2:9092,dbus-n3:9092
acks=1
retries=3
batch.size=16384
linger.ms=1
buffer.memory=33554432
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer

```

修改/app/dbus/dbus-heartbeat-0.4.0/conf/jdbc.properties内容如下：

```
DB_TYPE=mysql
DB_KEY=dbus.conf
DB_DRIVER_CLASS=com.mysql.jdbc.Driver
# db-dbusmgr是已安装mysql服务所在机器的IP
DB_URL=jdbc:mysql://db-dbusmgr:3306/dbusmgr?characterEncoding=utf-8
DB_USER=dbusmgr
# 将(#password_place_holder#}替换成/app/dbus/DBus-0.4.0.sql中dbusmgr用户的密码
DB_PWD=(#password_place_holder#}
DS_INITIAL_SIZE=1
DS_MAX_ACTIVE=1
DS_MAX_IDLE=1
DS_MIN_IDLE=1
```

修改/app/dbus/dbus-heartbeat-0.4.0/conf/stat_config.properties内容如下：

```
dbus.statistic.topic=dbus_statistic
kafka.offset=none
influxdb.url=http://dbus-n1:8086
influxdb.dbname=dbus_stat_db
influxdb.tablename=dbus_statistic
```

在dbus-n3上的安装和dbus-n2上的步骤相同

## 7.4 启动

在dbus-n2上进入/app/dbus/dbus-heartbeat-0.4.0目录执行如下命令：

```
# 赋予heartbeat.sh可执行权限
chmod 744 heartbeat.sh
# 启动心跳
./heartbeat.sh &
```

在dbus-n3上的安装和dbus-n2上的步骤相同




# 8. 安装Grafana

## 8.1 下载

推荐下载grafana版本：grafana-3.1.1
地址：[https://grafana.com/grafana/download](https://grafana.com/grafana/download)

## 8.2 安装

在dbus-n1上首先切换到root用户，执行如下命令

```
rpm -ivh grafana-3.1.1-1470047149.x86_64.rpm
```

## 8.3 配置

在dbus-n1上修改配置文件/etc/grafana/grafana.ini的[decurity]部分如下，其它部分不用修改：

```
[security]
# default admin user, created on startup
admin_user = admin

# default admin password, can be changed before first start of grafana,  or in profile settings
admin_password = admin
```

## 8.4 启动

在dbus-n1上执行如下命令：

```
service grafana-server start
```

## 8.5 验证

### 8.5.1 登录grafana

打开浏览器输入：http://dbus-n1:3000/login，出现如下页面：

用户名：admin 

密码：admin

![](img/install-base-components-05.png)

### 8.5.2 配置grafana

#### 8.5.2.1 配置Grafana influxdb数据源如下图：

![](img/install-base-components-06.png)

![](img/install-base-components-07.png)

密码：dbus!@#123 (安装influxdb初始化配置脚本设置的密码)

#### 8.5.2.2 配置Grafana Dashboard

下载Schema Dashboard配置：initScript/init-table-grafana-config/grafana-schema.cfg

下载Table Dashboard配置：initScript/init-table-grafana-config/grafana-table.cfg

下载log Dashboard 配置：init-scripts/init-log-grafana-config/*.cfg

操作步骤如下：

![](img/install-base-components-08.png)

![](img/install-base-components-09.png)

分别上传schema.json和table.json的配置文件

![](img/install-base-components-10.png)

导入后出现如上图所示的两个dashboards。