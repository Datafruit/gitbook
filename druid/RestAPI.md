RestAPI使用说明
==============================  

集群所有的数据、任务管理和数据查询都是以RestAPI的方式进行，下文总结相应的说明。  

## 数据管理  

所有的数据都是由Coordinator节点管理，所以所有与数据相关的管理请求都发送到该节点。  

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


 
