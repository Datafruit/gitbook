History配置说明
==============================

History下载实时task上传的文件，并加载到内存提供查询服务。  

History的配置文件包括两部分内容：jvm.config和runtime.properties。 

## JVM配置  
>`安装目录`/conf/druid/Broker/jvm.config配置内容如下：  

```
-server
-Xms4g
-Xmx4g
-XX:MaxDirectMemorySize=24g
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=var/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dlog.file.path=var/logs
-Dlog.file.type=historical
-Dcom.sun.management.jmxremote.port=2627
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
-Djava.rmi.server.hostname=192.168.0.212
-Ddruid.dir=/opt/apps/druidio_sugo

```

建议统一用UTC时区，java.io.tmpdir用于指定运行时临时文件目录，请确保有该目录的权限和存储足够存放服务的log。  

需要注意的是由于我们使用了自定义的log4j2，在此需要指定log存储位置和节点类型，分别对应`-Dlog.file.path`和`-Dlog.file.type`，
另外需要指定项目的Druid的安装目录`-Ddruid.dir`。 

## History节点属性配置  
>`安装目录`/conf/druid/history/runtime.properties配置内容如下：

```
# 服务名，一个集群多个为同一个服务名
druid.service=druid/historical
# 服务启动的机器ip或域名，此处需要配置正确，避免在zookeeper引起错乱
druid.host=192.168.0.212
# 服务的端口，默认即可
druid.port=8083

# 必须指定数据段的类型为lucene，否则无法使用索引
druid.historical.segment.type=lucene

# 查询并发线程数和每个线程需要的堆外内存
# 一般情况看机器资源情况，druid.processing.buffer.sizeBytes大小在500M-2G，druid.processing.numThreads为cpu核心数-1。
# 在测试环境中，数据段小，查询简单druid.processing.buffer.sizeBytes设置为100-200M也没有问题，生产环境建议设置500M以上。
# 配置时确保task的堆外内存 -XX:MaxDirectMemorySize > numThreads*sizeBytes
druid.processing.numThreads=11
druid.processing.buffer.sizeBytes=1073741824

# history节点希望分配的存储，这个值并不会强制限制history的存储大小，为coordinator在分配segment时提供参考。
druid.server.maxSize=300000000000
# history下载的数据段存储的位置，如果有多块硬盘，可以配置多个path，便于充分利用磁盘性能
druid.segmentCache.locations=[{"path":"var/druid/segment-cache","maxSize":300000000000}]

# 处理查询请求线程池 一般情况20~50
druid.server.http.numThreads=25

```

>关于内存的设置建议   
history节点利用堆外内存存储查询中间结果以及数据段的内存映射，堆外内存越大，就越能降低使用磁盘分页的可能性，查询的速度就越快。  
对外内存可以通过druid.processing.numThreads和druid.processing.buffer.sizeBytes控制查询内存消耗。而堆内存一般建议 250M*(processing.numThreads)。  
另外，Broker节点GC策略最好使用CMS。
