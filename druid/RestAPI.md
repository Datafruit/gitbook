RestAPI使用说明
==============================  

集群所有的数据、任务管理和数据查询都是以RestAPI的方式进行，下文总结相应的说明。  

## 数据管理  

所有的数据都是由Coordinator节点管理，所以所有与数据相关的管理请求都发送到该节点。  

* 查看所有的数据源
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources    
* 查看指定数据源信息
curl http://CoordinatorIP:8081/druid/coordinator/v1/datasources/{DatasourcName}  


