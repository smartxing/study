## 查看集群状态
    GET /_cat/health?v

    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
    1515807781 09:43:01  elasticsearch yellow          1         1     10  10    0    0       10             0                  -                 50.0%

    如何快速了解集群的健康状况？green、yellow、red？
    green：每个索引的primary shard和replica shard都是active状态的
    yellow：每个索引的primary shard都是active状态的，但是部分replica shard不是active状态，处于不可用的状态
    red：不是所有索引的primary shard都是active状态的，部分索引有数据丢失了

    所以说 我们单机启动的状态是yellow

## shard&replica
    （1）index包含多个shard
    （2）每个shard都是一个最小工作单元，承载部分数据，lucene实例，完整的建立索引和处理请求的能力
    （3）增减节点时，shard会自动在nodes中负载均衡
    （4）primary shard和replica shard，每个document肯定只存在于某一个primary shard以及其对应的replica shard中，不可能存在于多个primary shard
    （5）replica shard是primary shard的副本，负责容错，以及承担读请求负载
    （6）primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改
    （7）primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard
    （8）primary shard不能和自己的replica shard放在同一个节点上（否则节点宕机，primary shard和副本都丢失，起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上

## 如何分配 shard&replica

    （1）primary&replica自动负载均衡，6个shard，3 primary，3 replica
    （2）每个node有更少的shard，IO/CPU/Memory资源给每个shard分配更多，每个shard性能更好
    （3）扩容的极限，6个shard（3 primary，3 replica），最多扩容到6台机器，每个shard可以占用单台服务器的所有资源，性能最好
    （4）超出扩容极限，动态修改replica数量，9个shard（3primary，6 replica），扩容到9台机器，比3台机器时，拥有3倍的读吞吐量
    （5）3台机器下，9个shard（3 primary，6 replica），资源更少，但是容错性更好，最多容纳2台机器宕机，6个shard只能容纳0台机器宕机

## es 一致性解决 乐观锁（_version）

    第创建一个document的时候，它的_version内部版本号就是1；以后，每次对这个document执行修改或者删除操作，都会对这个_version版本号自动加1；哪怕是删除，也会对这条数据的版本号加1