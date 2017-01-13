MiddleManager配置说明
==============================

MiddleManager相对来说非常轻量级，只是单纯的以JVM进程的方式调起需要执行的task。
但具体的task进程，特别是写索引的进程(我们一般可以叫做实时task)，不仅要写索引还要提供查询服务，需要消耗一定系统资源。
task运行的jvm参数可以通过修改`druid.indexer.runner.javaOpts`调整。
建议实时task一般写的数据段数据量控制在1-2千万左右，这是一个平衡值，过多单个数据段查询慢，过少会导致数据段过多，都会在一定程度上影响查询性能。
另外，每个task的内存2G，堆外内存2G，一台机器的内存 > task数*(2G+2G)。  

MiddleManager的配置文件包括两部分内容：jvm.config和runtime.properties。  

## JVM配置  
>`安装目录`/conf/druid/MiddleManager/jvm.config配置内容如下：

```
-server
-Xms64m
-Xmx64m
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=/home/druid/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dlog.file.path=/data2/druidio/logs/jvm
-Dlog.file.type=middleManager
-Ddruid.dir=/opt/apps/druidio_sugo
-Dlog.configurationFile=/opt/apps/druidio_sugo/conf/druid/_common/log4j2-default.xml

```

该节点不会消耗过多内存，一般情况64~128M内存足够。建议统一用UTC时区，java.io.tmpdir用于指定运行时临时文件目录，请确保有该目录的权限和存储足够存放服务的log。  

需要注意的是由于我们使用了自定义的log4j2，在此需要指定log存储位置和节点类型，分别对应`-Dlog.file.path`和`-Dlog.file.type`，
另外需要指定项目的Druid的安装目录`-Ddruid.dir`。  

为了区分task的配置需要指定`-Dlog.configurationFile`，否则task日志会出现问题。  

## MiddleManager节点属性配置  
>`安装目录`/conf/druid/coordinator/runtime.properties配置内容如下：  

```
# 服务名，一个集群多个为同一个服务名
druid.service=druid/middleManager

# 服务启动的机器ip或域名，此处需要配置正确，避免在zookeeper引起错乱
druid.host=192.168.0.212

# 服务的端口，默认即可
druid.port=8091

# MiddleManager最大启动的task进程数
druid.worker.capacity=12

# MiddleManager查询并发线程数和每个线程需要的堆外内存，配置时确保task的堆外内存 -XX:MaxDirectMemorySize > numThreads*sizeBytes
druid.processing.numThreads=3
druid.processing.buffer.sizeBytes=524288000

# 每个task进程的jvm配置，MaxDirectMemorySize表示堆外内存
druid.indexer.runner.javaOpts=-server -Xmx2g -XX:MaxDirectMemorySize=2g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager -Djava.io.tmpdir=var/tmp

# task运行时临时目录
druid.indexer.task.baseTaskDir=var/druid/task
# hadoop任务所需要的hadoop版本，指定前确保版本所对应的目录已经存在于druid.extensions.hadoopDependenciesDir
druid.indexer.task.defaultHadoopCoordinates=["org.apache.hadoop:hadoop-client:2.7.2"]
# hadoop task临时目录
druid.indexer.task.hadoopWorkingPath=var/druid/hadoop-tmp

# 用于处理http请求的线程数
druid.server.http.numThreads=25

```

