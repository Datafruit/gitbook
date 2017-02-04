Supervisor例子
=================================

Druid在0.9.0之后提供了Supervisor的功能，对于挂掉的task可以重新拉起，目前其只实现了kafka的对接，我们也扩展了与metaq的对接。目前，我们支持kafka、metaq消息的不丢不重，且数据延迟也能正常接入。一个[Supervisor的定义](lucene_supervisor.json)：  

```
{
  "type": "lucene_supervisor",                       #`lucene_supervisor`为我们扩展的supervisor类型，
  "dataSchema": {
    "dataSource": "test",                            #数据源名
    "parser": {
      "type": "string",                              
      "parseSpec": {
        "format": "url",                             #数据解析方式，url为我们扩展的解析，支持url解析，另外还支持json/csv/tsv/正则等
        "timestampSpec": {
          "column": "EventDateTime",                 #基于时序的存储，需要指定时间字段
          "format": "millis"                         #format为日期的格式，其中millis=毫秒数，posix=秒数，另外我们扩展了此处，可以指定所需的format和时区
        },
        "dimensionsSpec": {
          "dimensions": [                            #指定需要索引的字段，我们扩展了此处，支持多种数据类型，且可以支持多值
            {
              "name": "IP",
              "type": "string"
            },
            {
              "name": "Province",
              "type": "string"
            },
            {
              "name": "Value",
              "type": "float"
            },
            {
              "name": "age",
              "type": "int"
            },
            {
              "name": "length",
              "type": "long"
            },
            {
              "name": "time",
              "type": "date",
              "format": "yyyy-MM-dd HH:mm",
              "timeZone": "+08:00"
            }
          ],
          "dimensionExclusions": [],
          "spatialDimensions": []
        }
      }
    },
    "metricsSpec": [],
    "granularitySpec": {
      "type": "uniform",
      "segmentGranularity": "DAY",                  #每个数据段的粒度，可以根据实际情况设置，可以设置为HOUR/DAY
      "queryGranularity": "NONE"
    }
  },
  "tuningConfig": {
    "type": "kafka",                                #指定数据来源
    "maxRowsPerSegment": 20000000,                  #每个段最大行数，到达后将生成新的段
    "basePersistDirectory": "/opt/apps/druidio_sugo/var/tmp/taskStorage",      #task临时目录
    "buildV9Directly": true
  },
  "ioConfig": {
    "topic": "test",                                 #kafka对应的topic
    "consumerProperties": {
      "bootstrap.servers": "192.168.0.217:9092,192.168.0.215:9092"     #kafka的broker位置
    },
    "taskCount": 2,                                  #启动的task数
    "replicas": 1,                                   #task的replicas数
    "taskDuration": "P1D",                           #task执行的时间，一般情况根据数据情况可以设置为 P1D  PT1H或PT2H等
    "useEarliestOffset": "true"                      #第一次消费是否从最早的位置开始消费
  }
}

```
curl -X POST -H 'Content-Type: application/json' -d @supervisor-spec.json http://overlord:port/druid/indexer/v1/supervisor  
即可启动supervisor，相应的任务可以在http://overlord:port 页面看到具体的执行和日志。
关闭所有task：curl -X POST -H 'Content-Type: application/json' http://overlord:port/druid/indexer/v1/supervisor/数据源名/shutdown
