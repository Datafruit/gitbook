
Overlord配置说明
==============================

Overlord的主要功能是管理task任务，所有task都以http post的方式发送给Overlord。Overlord有本地(local)和集群(remote)两种模式，当使用本地模式时，Overlord还承担这进程调度的职责，Overlord会在本地启动进程，需要注意的是需要Middlemanager相关的配置，才能启动本地模式。生产环境需要使用集群模式，在集群模式中，Overlord只会分配task给相应的Middlemanager。  

Overlord的配置文件包括两部分内容：jvm.config和runtime.properties。  

## JVM配置  
>`安装目录`/conf/druid/overlord/jvm.config配置内容如下：  

```
-server
-Xms1g
-Xmx1g
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=var/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dlog.file.path=var/logs
-Dlog.file.type=overlord
-Ddruid.dir=/opt/apps/druidio_sugo

```

该节点不会消耗过多内存，一般情况2-4G内存足够。建议统一用UTC时区，java.io.tmpdir用于指定运行时临时文件目录，请确保有该目录的权限和存储足够存放服务的log。  
需要注意的是由于我们使用了自定义的log4j2，在此需要指定log存储位置和节点类型，分别对应`-Dlog.file.path`和`-Dlog.file.type`，
另外需要指定项目的Druid的安装目录`-Ddruid.dir`。  

## Overlord节点属性配置  
>`安装目录`/conf/druid/overlord/runtime.properties配置内容如下：  

```
# Overlord节点的服务名，一个集群多个为同一个服务名
druid.service=druid/overlord

# 服务启动的机器ip或域名，此处需要配置正确，避免在zookeeper引起错乱
druid.host=192.168.0.220

# Overlord服务的端口，默认即可
druid.port=8090

# 在Overlord启动一段时间后，才开始管理task的队列。进程刚启动时，需要一段时间来协调网络。
druid.indexer.queue.startDelay=PT30S

# Druid有两种模式：local和remote，分布式环境需要设置为remote
druid.indexer.runner.type=remote

# Druid的task信息存储有两种模式：local和metadata，local为内存存储，metadata为数据库存储，生产环境使用metadata
druid.indexer.storage.type=metadata

```

