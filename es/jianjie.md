 ### 谁在使用
     Github Github使用Elasticsearch搜索20TB的数据，包括13亿的文件和1300亿行的代码，Github在2013年1月升级了他们的代码搜索，由solr转为elasticsearch
     Foursquare  实时搜索5千万地点信息？Foursquare每天都用Elasticsearch做这样的事
     SoundCloud  SoundCloud使用Elasticsearch来为1.8亿用户提供即时精准的音乐搜索服务
     Fog Creek  Elasticsearch使Fog Creek可以在400亿行代码中进行一个月3千万次的查询
     StumbleUpon    Elasticsearch是StumbleUpon的关键部件，它每天为社区提供百万次的推荐服务
     Sony Sony公司使用elasticsearch 作为信息搜索引擎
     
 ### 倒排索引
    比如数据库里面有  
     
     1   广发银行信用卡电子账单
     2   浦发银行信用卡电子账单
     3   手机2月份账单
     
     我们建立倒排索引就是 先对内容进行分词
     账单  1，2，3
     广发  1
     银行  1，2
     手机  3 
     
     如果我们在搜 广发账单， 我们对其分词 广发，账单 可以把相关的都找出来 这个也叫做全文检索
     
 ### es 特性
    1 自动维护数据分布到多个集群点上
    2 自动维护冗余备份
    3 高级api 灵活访问
     
     
 ### 一些概念先过一下 
   #### 和数据库对比
     Index               (数据库)
     Type                (表)
     Document            (行)
     Field               (列)
     Id                  (主键/唯一键)
   #### 基础概念 
     Cluster
           集群,分布式的基础,其中有多个节点,去中心化,跟任何一个节点通信与跟整、集群通信是一样的
     Node
           节点,集群中的每台物理机器,分主节点,数据节点等
     Index
           数据集合 
     Type
            数据的逻辑分类(基于应用场景)
     Document
            数据的最小单元,一般以Json格式描述
     Shards
           索引分片,可以把一个大的索引拆分成多个,分布到不同的节点上,构成分布式搜索
     Replicas
            索引副本, 作用之一是可以提高系统容错性,当某个分片损坏或丢失时可以从副本中恢复,另一个作用是要提高es的查询效率,会自动对搜索请求进行负载均衡
            
            
     Recovery
            数据重新分布,在有节点加入或退出时会根据机器的负载对索引分片进行重新分配,挂掉的节点重新启动时也会进行数据恢复
     River
            es的一个数据源,其它存储方式(比如数据库)同步数据到es的一个方法
     Gateway
            es索引的持久化存储方式,默认索引先存放到内存,内存满了在持久化到硬盘,es集群关闭再重新启动时会从gateway读取索引数据.支持多种类型gateway:本地文件系统,分布式文件系统,HDFS等
     discovery.zen
            自动发现节点机制,通过广播协议寻找节点以及进行节点之间通信
     Transport
             内部节点或集群与客户端的交互方式,默认使用TCP
               
       