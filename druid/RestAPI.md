RestAPI使用说明
==============================  

集群所有的数据、任务管理和数据查询都是以RestAPI的方式进行，下文总结相应的说明。  

## 数据管理  

所有的数据都是由Coordinator节点管理，所以所有与数据相关的管理请求都发送到该节点。  

### 数据源管理  

* 查看所有的数据源  
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources    

* 查看指定数据源信息  
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}  

* 屏蔽数据源  
curl -X DELETE -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}   

* 恢复数据源  
curl -X POST -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}    

* 查看数据源所有时间段
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/intervals  

* 查看指定时间段的数据段(interval的格式：2017-03-24T01:00:00.000Z_2017-03-24T02:00:00.000Z)  
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/intervals/{interval}  

* 查看指定时间段的数据段的服务视图(interval的格式：2017-03-24T01:00:00.000Z_2017-03-24T02:00:00.000Z)  
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/intervals/{interval}/serverview    

* 清理指定时间段的数据段(interval的格式：2017-03-24T01:00:00.000Z_2017-03-24T02:00:00.000Z)  
curl -X DELETE -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/intervals/{interval}   

* 查看数据源所有的数据段   
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/segments   

* 查看数据源指定数据段信息  
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/segments/{segmentId}  

* 删除数据源指定数据段  
curl -X DELETE -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/segments/{segmentId}  

* 恢复数据源指定数据段   
curl -X POST -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}/segments/{segmentId}   

### 元数据管理  

* 查看所有的数据源  
curl http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources  
`注意`  可以通过参数full查看所有数据源的明细，通过参数includeDisabled指定是否查看的数据源包括已经屏蔽的数据源，例如：curl http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources?full=true&includeDisabled=false    
* 查看指定数据源的元数据信息，包括数据段信息  
curl http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources/{DatasourcName}  
* 查看指定数据源的数据段信息  
curl http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources/{DatasourcName}/segments  
`注意`  可以通过参数full查看所有数据源的明细, 例如：curl http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources/{DatasourcName}/segments?full=true  
* 查看指定数据源可供查询的数据段  
curl -X POST -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources/{DatasourcName}/segments  
`注意`  可以通过参数full查看所有的明细, 例如：curl -X POST -H 'Content-Type: application/json' http://CoordinatorIP:8081/druid/coordinator/v1/metadata/datasources/{DatasourcName}/segments?full=true  
 
### 规则管理  

集群数据的加载、删除都可以制定相应的Rule。类似于最近一个月数据加载到hot节点，只保留最近半年的数据，都可以通过定义Rule来实现。  

* 查看所有的Rule  
curl http://CoordinatorIP:8081/druid/coordinator/v1/rules  
* 查看指定数据源的Rule  
curl http://CoordinatorIP:8081/druid/coordinator/v1/rules/{DatasourcName}  
* 设置数据源的Rule  
curl -X POST -H 'Content-Type: application/json' -d @rule.json http://CoordinatorIP:8081/druid/coordinator/v1/rules/{DatasourcName}  
rule.json可以定义json，具体的规则的格式可参考[druid社区Rule](http://druid.io/docs/latest/operations/rule-configuration.html)  

* 查看数据源Rule的修改历史记录  
curl http://CoordinatorIP:8081/druid/coordinator/v1/rules/{dataSourceName}/history  


## 任务管理

### Task相关的管理  

* 关闭task Id指定的task    
- curl -X 'POST' -H 'Content-Type: application/json' http://overlord IP:8090/druid/indexer/v1/task/{task Id}/shutdown   

* 获取处于waiting状态的task信息  
- curl http://overlord IP:8090/druid/indexer/v1/waitingTasks  

* 获取处于pending状态的task信息  
- curl http://overlord IP:8090/druid/indexer/v1/pendingTasks  

* 获取处于running状态的task信息  
- curl http://overlord IP:8090/druid/indexer/v1/runningTasks  

* 获取已完成任务的task信息  
- curl http://overlord IP:8090/druid/indexer/v1/completeTasks  

* 获取指定task(task Id)运行日志信息  
- curl http://overlord IP:8090/druid/indexer/v1//task/{taskid}/log  


### Worker相关的管理  
* 获取worker行为信息  
- curl http://overlord IP:8090/druid/indexer/v1/worker    

* 设置worker授权信息  
- curl -X curl -H 'Content-Type: application/json' -d @workerBehaviorConfig.json -d "X-Druid-Author={配置信息}&X-Druid-Comment={配置信息}" http://overlord IP:8090/druid/indexer/v1/worker  

```
公平调用Middlemanager资源的workerBehaviorConfig.json格式为：
{
	"selectStrategy": {
	   "type":"equalDistribution"
	 }
}
```

* 获取worker历史信息  
- curl http://overlord IP:8090/druid/indexer/v1/worker/history  

* 获取worker运行信息  
- curl http://overlord IP:8090/druid/indexer/v1/workers  

### supervisor相关的管理  

* 根据supervisorSpec.json配置信息创建supervisor服务  
- curl -X curl -H 'Content-Type: application/json' -d @supervisorSpec.json http://overlord IP:8090/druid/indexer/v1/supervisor  
```
说明：  
1、如果相同数据源的supervisor已经存在，则会导致正在运行的supervisor会通知其管理的所有任务终止读取并执行segment发布。
2、推出正在执行的supervisor。
3、使用Request请求体内的规范创建一个新的supervisor，它会接管处于发布状态的任务，已经创建新的任务从处于发布状态的任务的介绍offset处开始读取数据。
```

* 获取当前活跃的supervisor列表  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor  

* 获取当前活跃的supervisor列表  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor/{supervisorId}  

* 重置消费位置  
- curl -X 'POST' http://overlord IP:8090/druid/indexer/v1/supervisor/{supervisorId}/reset   

* 获取指定supervisorId的状态  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor/{supervisorId}/status  

* 关闭supervisorId指定的supervisor，同时使其管理的所有任务终止读取，进入segment发布状态  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor/{supervisorId}/shutdown  

* 查看所有supervisor的历史  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor/history  

* 获取指定supervisor(supervisor Id)的历史  
- curl http://overlord IP:8090/druid/indexer/v1/supervisor/{supervisor Id}/history    


## 节点信息  

### Coordinator  

* 查看Coordinator的leader  
curl http://CoordinatorIP:8081/druid/coordinator/v1/leader  

* 查看数据加载情况  
curl http://CoordinatorIP:8081/druid/coordinator/v1/loadstatus  
`注意`  可以通过参数full查看各个数据源replicas的情况，例如：curl http://CoordinatorIP:8081/druid/coordinator/v1/loadstatus?full=true  

* 查看正在加载或者删除的数据段  
curl http://CoordinatorIP:8081/druid/coordinator/v1/loadqueue  
`注意`  可以通过参数full查看所有的明细, 例如：可以通过参数full查看各个数据源replicas的情况，例如：curl http://CoordinatorIP:8081/druid/coordinator/v1/loadqueue?full=true  

* 查看Coordinator动态配置的参数  
curl http://CoordinatorIP:8081/druid/coordinator/v1/config  

* 设置Coordinator动态配置的参数  
curl -X POST -H 'Content-Type: application/json' -d @config.json http://CoordinatorIP:8081/druid/coordinator/v1/config  
config.json可以配置为：
```
{"millisToWaitBeforeDeleting":900000,"mergeBytesLimit":524288000,"mergeSegmentsLimit":100,"maxSegmentsToMove":5,"replicantLifetime":15,"replicationThrottleLimit":10,"balancerComputeThreads":1,"emitBalancingStats":false,"killDataSourceWhitelist":null}  
每个参数的解释：
millisToWaitBeforeDeleting  删除数据段等待的时长，目的是为了待数据段稳定后删除，避免在balance的过程中存在删除恢复的情况  
mergeBytesLimit 自动合并数据段时，只合并小于此值的段，忽略，默认即可  
mergeSegmentsLimit 每个task合并最多的段，忽略，默认即可  
maxSegmentsToMove  每次balance的时候最多允许移动的数据段数  
replicantLifetime  数据段replicas的时候，超过这个时间范围还没有replicat完成，会出现没有replicas的警告  
replicationThrottleLimit  每次允许最多replicat多少个数据段  
balancerComputeThreads 做balance的线程数，如果不想做balance可以设置为0，可以结合replicas为1的时候使用，避免数据balance的时候查询不准确  
emitBalancingStats  是否统计balance的情况，一般建议关闭  
killDataSourceWhitelist  不需要的datasource白名单，但task需要落地白名单中的datasource时，如果`druid.coordinator.kill.on`为true，则会停止这些task
```  

### Overlord  

* 查看Overlord的leader  
curl http://overlordIP:8090/druid/indexer/v1/leader  

## 查询API  

待完善
