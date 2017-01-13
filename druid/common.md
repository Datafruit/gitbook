Druid配置参数说明  
=================================== 

## 1.通用配置

>`安装目录`/conf/druid/_common/common.runtime.properties文件配置内容如下：

```
# 日志模式，logging表示单纯的文件日志，ingest表示上报消费的数据情况，用于数据质量监控
# 若使用ingest，必须指定druid.emitter.ingest.recipientBaseUrl，用于接收数据
druid.emitter=logging
#druid.emitter=composing
#druid.emitter.composing.emitters=["logging", "ingest"]
#druid.emitter.ingest.recipientBaseUrl=http://192.168.0.218:8080/api/v1/datapoints

#日志级别，默认info即可
druid.emitter.logging.logLevel=info
#是否开启启动时的包加载情况日志，默认即可
druid.startup.logging.logProperties=true

#Druid扩展依赖的目录，对应druid的`安装目录`/extensions
druid.extensions.directory=/opt/apps/druidio_sugo/extensions
druid.extensions.hadoopDependenciesDir=/opt/apps/druidio_sugo/hadoop-dependencies

#Druid可以定义不能呢个功能模块，默认使用postgresql数据库存储元数据，  
#如果更换mysql，可将`postgresql-metadata-storage`更换为`mysql-metadata-storage`
druid.extensions.loadList=["druid-kafka-eight", "postgresql-metadata-storage", "druid-hdfs-storage", "druid-lucene-extensions"]

#实时task运行完后log存储类型，如果需要hdfs统一保存可设置为`hdfs`
druid.indexer.logs.type=file
#实时task运行完后log上传文件位置
druid.indexer.logs.directory=/data1/druid/indexing-logs

#元数据保存数据库，如果是mysql数据库，`postgresql`改为`mysql`，并修改connectURI。  
#但需要将druid.extensions.loadList中的`postgresql-metadata-storage`更换为`mysql-metadata-storage`
druid.metadata.storage.type=postgresql
druid.metadata.storage.connector.connectURI=jdbc:postgresql://192.168.0.210:5432/druid_test
druid.metadata.storage.connector.password=druid
druid.metadata.storage.connector.user=druid

#设置管理服务的名称，默认即可
druid.selectors.coordinator.serviceName=druid/coordinator
druid.selectors.indexing.serviceName=druid/overlord

#设置需要监控的内容，druid包含多种监控，可根据需要配置
druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]

# Druid的实时task运行完后，数据备份位置，可以设置为hdfs，对应hdfs路径
druid.storage.type=local
druid.storage.storageDirectory=/data1/druid/storage

# Druid依赖与zookeeper同步信息，需要设置zookeeper路径，  
# druid.zk.paths.base表示同步数据信息的路径，  
#druid.discovery.curator.path主要用于服务发现
druid.zk.paths.base=/druid_test
druid.discovery.curator.path=/druid_test/discovery
druid.zk.service.host=127.0.0.1:2181

#针对用户分群的场景，如果没有该场景可以忽略
druid.lookup.lru.cache.maxEntriesSize=50
#druid.lookup.lru.cache.expireAfterWrite=300
#druid.lookup.lru.cache.expireAfterAccess=300

```

## 2.log文件配置  

>Druid相关的log文件位于目录：`安装目录`/conf/druid/_common，文件分别为：log4j2-default.xml和log4j2.xml  

log4j2-default.xml为实时task任务的日志配置，与官方提供的log4j一致，未做调整。  
log4j2.xml为Druid每个服务所使用的日志配置。  

之所以单独配置Druid每个服务的日志，是因为官方并未给出具体的配置，随着Druid服务的运行日志越来越大，
并且每分钟metric的日志和正常的log混合在一起，难以区分。log4j2.xml的文件内容如下：  

```
<Configuration status="WARN">
    <Properties>
        <Property name="LOG_DIR">${sys:log.file.path}</Property>
    <Property name="LOG_NAME">${sys:log.file.type}</Property>
    </Properties>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
        </Console>
        <RollingRandomAccessFile name="DruidLog" fileName="${LOG_DIR}/${LOG_NAME}.log"  filePattern="${LOG_DIR}/${LOG_NAME}.%d{yyyy-MM-dd}.log">
            <PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
            </Policies>
            <!--日志保留7天-->
            <DefaultRolloverStrategy>
                <Delete basePath="${LOG_DIR}" maxDepth="1">
                    <IfFileName glob="${LOG_NAME}.*.log" />
                    <IfLastModified age="7d" />
                </Delete>
            </DefaultRolloverStrategy>
        </RollingRandomAccessFile>
            <RollingRandomAccessFile name="MetricLog" fileName="${LOG_DIR}/${LOG_NAME}-metric.log" filePattern="${LOG_DIR}/${LOG_NAME}-metric.%d{yyyy-MM-dd}.log">
            <PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
            </Policies>
            <!--日志保留7天-->
            <DefaultRolloverStrategy>
                <Delete basePath="${LOG_DIR}" maxDepth="1">
                    <IfFileName glob="${LOG_NAME}-metric.*.log" />
                    <IfLastModified age="7d" />
                </Delete>
            </DefaultRolloverStrategy>
        </RollingRandomAccessFile>
    </Appenders>
    <Loggers>
    	<!--metric相关的日志另外输出-->
        <logger name="com.metamx.emitter.core.LoggingEmitter" level="info" additivity="false">
                <appender-ref ref="MetricLog" />
        </logger>
        <Root level="info">
            <AppenderRef ref="DruidLog"/>
        </Root>
    </Loggers>
</Configuration>

```

