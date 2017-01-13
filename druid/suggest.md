Druid安装建议
==============================

计算机的三个核心资源为cpu、内存和存储。根据这三方面，针对Druid各种服务做以下建议：  

## 存储  

Druid的History节点相对来说比较占用存储，Middlemanager只需要少量存储，存储数据的临时写入，一旦实时task任务完成后，数据将会上传到分布式文件系统，移交查询给History后，本地临时写入的文件将会自动清除。其他节点基本不需要存储，对存储也没有要求，此处只介绍History节点和Middlemanager的存储建议。  

为了提升实时task数据写入和查询速度，可以有可能尽量使用SSD盘，加快数据写入速度。另外，我们扩展了数据写入，可以指定多块盘同时写数据，也可以通过加硬盘和增加task数提升数据写入速度。但是，实时task并不是特别消耗存储，所以Middlemanager节点不需要太大的存储。  

History需要比较大的存储，所有的索引数据History都需要从本地加载，另外如果机器资源充足，还是有必要做replicat的，可以提升查询并发。如果有可能使用SSD盘，能够大幅度提升查询性能。建议使用多块硬盘，也能提升数据查询性能。  

建议History和Middlemanager节点使用独立的系统盘，避免因为频繁读盘，导致系统运行慢，而引起恶性循环。

## 内存  

Overlord和Coordinator为管理节点，不需要消耗太多内存，一般2~4G足够，也不需要使用堆外内存。  

Middlemanager节点，只是单纯的调度进程，使用的资源非常少，一般64~128M足够。但是需要注意Middlemanager中配置的实时task的内存，实时task需要参看自身所需要写入的数据段数，一般情况设置堆内存2~3G，堆外2~3G，这些要根据具体情况调。  

Broker节点合并History的查询结果需要较多的堆内存，建议堆内存设置到20~30G。同时Broker处理groupby查询海上花需要消耗堆外内存，一般情况建议druid.processing.buffer.sizeBytes设置到500M以上，然后结合系统资源情况，设置druid.processing.numThreads，充分利用系统的资源。druid.processing.numThreads也不用设置过大，一般建议(cpu数-1)。  

History节点需要堆外内存存储中间结果和对索引数据做内存映射，一般来说，堆外内存越大，数据段基本都可以基于内存映射，不需要硬盘分页，查询速度越快。相对来说，堆内存不是特别消耗，一般建议堆内存设置为：250mb * (processing.numThreads) 。  

## CPU  

Broker、History和Middlemanager节点所在的机器，尽量使用较好的cpu，建议cpu核心数在12以上。  

## 服务器搭配建议  

Overlord和Coordinator用于管理集群的元数据，属于辅助功能，可以部署在同一台服务器，所需的服务器配置不用很高，建议：  

* 4 vCPUs  
* 12 GB RAM  
* 100 GB HDD  

History和Middlemanager可以部署在一台服务器，他们都是数据服务节点，这两个服务对cpu、内存和硬盘都有要求，如果使用SSD会更好，使用一般硬盘的话，建议多加几块盘，一般情况建议： 

* 12 vCPUs
* 64 GB RAM
* 12*500G HDD  

Broker主要合并查询结果，它的所有操作基本上都是基于内存的，对cpu和内存要求高，可以与查询的UI层部署在同一台机器，建议：  

* 12 vCPUs
* 64 GB RAM
* 100 GB HDD  

另外，Druid对zookeeper的依赖比较重，建议使用单独的zookeeper集群。

