segment合并例子
=================================

## 示例 
```
{
	"type": "lucene_merge",
	"dataSource": "wikipedia",
	"mergeGranularity": "DAY",
	"interval": "2017-03-01/2017-05-01"
}
```  
停止实时接入任务(避免新增数据对合并造成影响):curl -X POST -H 'Content-Type: application/json' http://overlord:port/druid/indexer/v1/supervisor/{supervisorId}/shutdown   
提交合并task：curl -X POST -H 'Content-Type: application/json' -d @merge-task-spec.json http://overlord:port/druid/indexer/v1/task    
