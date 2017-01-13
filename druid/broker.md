Broker配置说明
==============================

Broker为Druid的查询节点，通过zookeeper监听得到每个数据段所在的history节点或者实时task，当有查询请求时，Broker节点会将查询分发到具体的history和实时task查询，然后汇总查询返回的结果。  

Broker的配置文件包括两部分内容：jvm.config和runtime.properties。  

## JVM配置  
>`安装目录`/conf/druid/Broker/jvm.config配置内容如下：  

```
-server
-Xms20g
-Xmx20g
-XX:MaxDirectMemorySize=20g
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=var/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dlog.file.path=var/logs
-Dlog.file.type=broker
-Ddruid.dir=/opt/apps/druidio_sugo

```
建议统一用UTC时区，java.io.tmpdir用于指定运行时临时文件目录，请确保有该目录的权限和存储足够存放服务的log。  

需要注意的是由于我们使用了自定义的log4j2，在此需要指定log存储位置和节点类型，分别对应`-Dlog.file.path`和`-Dlog.file.type`，
另外需要指定项目的Druid的安装目录`-Ddruid.dir`。  

## Broker节点属性配置  
>`安装目录`/conf/druid/broker/runtime.properties配置内容如下：

```
# 服务名，一个集群多个为同一个服务名
druid.service=druid/broker
# 服务启动的机器ip或域名，此处需要配置正确，避免在zookeeper引起错乱
druid.host=192.168.0.212
# 服务的端口，默认即可
druid.port=8082

#--------------------
# 缓存相关的配置
#--------------------
# 不需要缓存的查询，默认"groupBy", "select"
druid.broker.cache.unCacheable=["groupBy", "select"]
# 缓存类型，支持local、memcached和hybrid三种模式，
# 其中local表示堆内存，实际为LinkedHashMap
# hybrid为混合缓存，混合缓存支持两级缓存，一般情况local作为一级缓存，memcached作为二级缓存，需要注意的是二级缓存大于一级缓存
druid.cache.type=local
# 设置缓存大小，byte为单位
druid.cache.sizeInBytes=200000000
# useCache和populateCache需要配合使用，一个为是否读缓存，一个为是否写入缓存。
# 在使用的时候，一般情况两者要么都为false，要么都为true
# 在大集群的时候这两项需要false，尽量让history合并结果，然后传到broker
druid.broker.cache.useCache=false
druid.broker.cache.populateCache=false

# Broker以http方式发送查询请求，此处为http线程池
# 可以根据机器资源情况调整，一般30~50
druid.broker.http.numConnections=30

# 查询并发线程数和每个线程需要的堆外内存
# 一般情况看机器资源情况，druid.processing.buffer.sizeBytes大小在500M-2G，druid.processing.numThreads为cpu核心数-1。
# 在测试环境中，数据段小，查询简单druid.processing.buffer.sizeBytes设置为100-200M也没有问题，生产环境建议设置500M以上。
# 配置时确保task的堆外内存 -XX:MaxDirectMemorySize > (numThreads+numMergeBuffers)*sizeBytes
druid.processing.numThreads=11
druid.processing.buffer.sizeBytes=1073741824
# 当使用groupByV2时，需要设置该参数
#druid.processing.numMergeBuffers=3

# Broker处理查询请求线程池 一般情况20~50
druid.server.http.numThreads=25

```

>关于内存的设置建议  
Broker节点需要合并所有段返回的结果，在段多的情况，消耗的内存相应也会多，另外查询场景也会影响内存。  
Druid的groupby查询需要堆外内存，topN查询大部分情况需要同时消耗堆内存和堆外内存，每个数据段返回的结果在druid中都是堆内存存储，而topN计算使用的是堆外内存，除此之外所有的查询都使用堆内存。  
当查询大部分集中在使用groupby的时候，可以通过druid.processing.numThreads和druid.processing.buffer.sizeBytes控制查询内存消耗。  
而当查询以堆内存的消耗为主或者各种查询都有时，需要注意适当调大堆内存，特别是段多的时候，需要合并所有段的返回结果。堆内存不建议设置过大，避免带来GC慢，消耗cpu等问题，一般建议堆内存20~30G。另外，Broker节点GC策略最好使用CMS。