Coordinator配置说明
==============================

Coordinator负责管理所有的数据段，数据段的加载、删除、副本的管理都由其统一调度协调。事实上，Coordinator在数据段的管理方面并未直接与history通信，而是将history需要加载的段写到zookeeper相应history加载队列的路径，history一旦发现，则会开始加载提供查询服务。  

Coordinator的配置文件包括两部分内容：jvm.config和runtime.properties。  

## JVM配置  
>`安装目录`/conf/druid/coordinator/jvm.config配置内容如下：  

```
-server
-Xms2g
-Xmx2g
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=var/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dlog.file.path=var/logs
-Dlog.file.type=coordinator
-Ddruid.dir=/opt/apps/druidio_sugo

```

该节点不会消耗过多内存，可根据数据段的数量适当调整内存，一般情况4-8G内存足够。
建议统一用UTC时区，java.io.tmpdir用于指定运行时临时文件目录，请确保有该目录的权限和存储足够存放服务的log。  
需要注意的是由于我们使用了自定义的log4j2，在此需要指定log存储位置和节点类型，分别对应`-Dlog.file.path`和`-Dlog.file.type`，
另外需要指定项目的Druid的安装目录`-Ddruid.dir`。  

## Overlord节点属性配置  
>`安装目录`/conf/druid/coordinator/runtime.properties配置内容如下：  

```
# 服务名，一个集群多个为同一个服务名
druid.service=druid/coordinator

# 服务启动的机器ip或域名，此处需要配置正确，避免在zookeeper引起错乱
druid.host=192.168.0.220

# 服务的端口，默认即可
druid.port=8081

# Druid本身存在一个问题，当history节点加载大量数据段时，需要花费较多的时间，
# 若此处配置的时间较短，history节点未能加载完数据段，coordinator则会重新分配
# 这些数据段给其他history节点，这点非常不友好。广东数果增加了数据段动态加载的功能，
# 可以避免这种情况
druid.coordinator.startDelay=PT30S

# coordinator检测数据段有限性周期
druid.coordinator.period=PT30S

```
