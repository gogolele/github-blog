title: kafka ops
date: 2015-06-24 10:18:22
tags: dev
catogories: Kafka
---

## Kafka Ops Command
version: 2.11_0.8.2.0
### start / stop kafka daemon
```shell
bin/kafka-server-start.sh -daemon conf/server.properties
bin/kafka-server-stop.sh
```
### create a topic
```shell
bin/kafka-topics.sh --zookeeper e3kafka01 --create EC_Inventory --partitions 20 --replication-factor 3
```
### list & describe
```shell
bin/kafka-topics.sh --zookeeper e3kafka01 --list
bin/kafka-topics.sh --zookeeper e3kafka01 --describe --topic EC_Inventory
```

### add partition
only support add partition
```shell
bin/kafka-topics.sh --zookeeper e3kafka01 --alter --topic EC_Inventory --partition 25
```
### modify configuration parameter
```shell
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 1 --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my-topic --config max.message.bytes=128000
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my-topic --deleteConfig max.message.bytes
```

### delete topic
```shell
bin/kafka-run-class.sh kafka.admin.TopicCommand --delete --topic IVT_InventoryIncrementLog --zookeeper e3kafka01:2181,e3kafka02:2181,e3kafka03:2181
bin/kafka-topics.sh --zookeeper e3kafka01 --delete --topic IVT_InventoryIncrementLog
```
note:
enable the delete.topic.enable=true in config/server.properties

mannully delete topic
  1. stop kafka daemon
  2. clean kafka log in partitions folders.
  3. delete the topic metadata zkCli.sh, rmr /brokers/mytopic
  4. start the kafka daemon

### delete data in a topic
```shell
./kafka-topics.sh --zookeeper localhost --alter --topic TimingAPI --config retention.ms=1   // delete data in 1 ms
./kafka-topics.sh --zookeeper localhost --alter --topic TimingAPI --config retention.ms=3600000 // restore the retention configuration
```

### reassign / move the partitions
  1. write a json to control the move
  2. use bin/kafka-reassign-partitions.sh to generate the placement json
  3. use bin/kafka-reassign-partitions.sh to execute
  4. use bin/kafka-reassign-partitions.sh to verify

```shell
[tw79@e3ecapp31 kafka-client]$ cat topic-move-test.json
{
  "topics": [{"topic":"EC_IPService_Log_Metrics"}],
  "version":1
}
[tw79@e3ecapp31 kafka-client]$ bin/kafka-reassign-partitions.sh --zookeeper e3kafka01 --topics-to-move-json-file topic-move-test.json --broker-list "0,1,2,3,4" --generate
Current partition replica assignment

{"version":1,"partitions":[{"topic":"EC_IPService_Log_Metrics","partition":2,"replicas":[3,4]},{"topic":"EC_IPService_Log_Metrics","partition":3,"replicas":[4,0]},{"topic":"EC_IPService_Log_Metrics","partition":0,"replicas":[1,2]},{"topic":"EC_IPService_Log_Metrics","partition":1,"replicas":[2,3]},{"topic":"EC_IPService_Log_Metrics","partition":4,"replicas":[0,1]}]}
Proposed partition reassignment configuration

{"version":1,"partitions":[{"topic":"EC_IPService_Log_Metrics","partition":2,"replicas":[1,2]},{"topic":"EC_IPService_Log_Metrics","partition":3,"replicas":[2,3]},{"topic":"EC_IPService_Log_Metrics","partition":0,"replicas":[4,0]},{"topic":"EC_IPService_Log_Metrics","partition":1,"replicas":[0,1]},{"topic":"EC_IPService_Log_Metrics","partition":4,"replicas":[3,4]}]}
[tw79@e3ecapp31 kafka-client]$ cat /tmp/topic-move.json
{"version":1,"partitions":[{"topic":"EC_IPService_Log_Metrics","partition":2,"replicas":[1,2]},{"topic":"EC_IPService_Log_Metrics","partition":3,"replicas":[2,3]},{"topic":"EC_IPService_Log_Metrics","partition":0,"replicas":[4,0]},{"topic":"EC_IPService_Log_Metrics","partition":1,"replicas":[0,1]},{"topic":"EC_IPService_Log_Metrics","partition":4,"replicas":[3,4]}]}
[tw79@e3ecapp31 kafka-client]$ bin/kafka-reassign-partitions.sh --zookeeper e3kafka01 --reassignment-json-file /tmp/topic-move.json --execute
Current partition replica assignment

{"version":1,"partitions":[{"topic":"EC_IPService_Log_Metrics","partition":2,"replicas":[3,4]},{"topic":"EC_IPService_Log_Metrics","partition":3,"replicas":[4,0]},{"topic":"EC_IPService_Log_Metrics","partition":0,"replicas":[1,2]},{"topic":"EC_IPService_Log_Metrics","partition":1,"replicas":[2,3]},{"topic":"EC_IPService_Log_Metrics","partition":4,"replicas":[0,1]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions {"version":1,"partitions":[{"topic":"EC_IPService_Log_Metrics","partition":0,"replicas":[4,0]},{"topic":"EC_IPService_Log_Metrics","partition":4,"replicas":[3,4]},{"topic":"EC_IPService_Log_Metrics","partition":2,"replicas":[1,2]},{"topic":"EC_IPService_Log_Metrics","partition":1,"replicas":[0,1]},{"topic":"EC_IPService_Log_Metrics","partition":3,"replicas":[2,3]}]}
[tw79@e3ecapp31 kafka-client]$ bin/kafka-reassign-partitions.sh --zookeeper e3kafka01 --reassignment-json-file /tmp/topic-move.json --verify
Status of partition reassignment:
Reassignment of partition [EC_IPService_Log_Metrics,0] completed successfully
Reassignment of partition [EC_IPService_Log_Metrics,4] completed successfully
Reassignment of partition [EC_IPService_Log_Metrics,2] completed successfully
Reassignment of partition [EC_IPService_Log_Metrics,1] completed successfully
Reassignment of partition [EC_IPService_Log_Metrics,3] completed successfully
```

### produer console / consumer console
```shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic tes
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --group pv
```
### offset consumer group
```shell
bin/kafka-run-class.sh kafka.tools.ExportZkOffsets
bin/kafka-run-class.sh kafka.tools.UpdateOffsetsInZK
bin/kafka-run-class.sh kafka.tools.UpdateOffsetsInZK earliest config/consumer.properties  page_visits

```
### ZK data Watch
Broker Node Registry

/brokers/ids/[0...N] --> host:port (ephemeral node)
Broker Topic Registry

/brokers/topics/[topic]/[0...N] --> nPartions (ephemeral node)
Consumer Id Registry

/consumers/[group_id]/ids/[consumer_id] --> {"topic1": #streams, ..., "topicN": #streams} (ephemeral node)
Consumer Offset Tracking

/consumers/[group_id]/offsets/[topic]/[broker_id-partition_id] --> offset_counter_value ((persistent node)
Partition Owner registry

/consumers/[group_id]/owners/[topic]/[broker_id-partition_id] --> consumer_node_id (ephemeral node)


reference
http://kafka.apache.org/documentation.html#operations
http://www.cnblogs.com/fxjwind/p/3794495.html

-EOF-

Tao Wu@CA, USA
