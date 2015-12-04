title: ElasticSearch运维点滴
date: 2014-10-20 10:10:52
categories: dev
tags: ElasticSearch
---

ElasticSearch在节点有不可用的情况下，可能产生不一致的情况，这时候需要进行一些处理。

如果ElasticSearch节点不正常，通过head，可以看到ElasticSearch的状态是红色或者黄色。

在我运维的过程中，一个DataCenter的两个节点crash，后来重新安装磁盘，将节点up。但是这两个
节点上的数据已经丢失。我们先在这个两个节点重新部署ElasticSearch，并启动。
配置方面，主要是如下地方需要修改
```
cluster.name: XXX
node.name: "node1"
node.DC: dc1
index.number_of_shards: 2
index.number_of_replicas: 1
network.bind_host: 192.0.0.1
transport.tcp.port: 9300
http.port: 9200
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["192.0.0.2:9300", "192.0.0.3:9300", "192.0.0.3:9300"]
#indices.cache.filter.size: 20%
indices.cache.filter.size: 512000000
indices.cache.filter.expire: 10m
#index.cache.field.max_size: 1024000000
#index.cache.field.expire: 10m
indices.fielddata.cache.size: 1024000000
indices.fielddata.cache.expire: 10m
#index.cache.field.type: soft
discovery.zen.fd.ping_timeout: 120s
discovery.zen.fd.ping_retries: 1000
discovery.zen.minimum_master_nodes: 2
discovery.zen.fd.ping_interval: 30s
```

配置完毕后启动ElasticSearch
```shell
  ./elasticsearch -p es.pid
```
出现too many openfiles错误。
```shell
   vim /etc/security/limits.conf
```
在文件末尾加入两行
```
tony soft nofile 1048576
tony hard nofile 1048576
```


启动后发现节点已经加入集群，但是有的index处于unassigned的状态，此时我们可以使用如下命令，将
unassigned的index加入
```
curl -X POST \
'http://localhost:9200/_cluster/reroute?pretty=true' \
-d '{ "commands" : [ { "allocate" : { "index" : "e4_logstash-2014.10.17", "shard" : 0 , "node" : "LOGX-ES-E4Flm01", "allow_primary" : 1 }}]}'
```

后来又出现一个DC的某个节点不可用的情况，这时候也会出现index unassigned，但是我们已经没有节点
可以放置这个index的replica，这种情况下，我们可以将replica减小，使集群恢复正常。
```
curl -XPUT 'http://172.16.21.225:9200/e3_logstash-2014.10.17/_settings' -d '
{
"index" : {
"number_of_replicas" : 0
} }
'
```
另外，我们还有duplicate一个index的需求，此时，我们可以通过curl命令去获取到当前index的schema，
再创建。

这个命令可以看到整个cluster所有index的metadata
```
curl -XGET 'http://172.16.21.225:9200/_cluster/state'
```
这个命令可以看到一个index的基本设置，
```
curl -XGET 'http://172.16.21.225:9200/e3_logstash-2014.10.19/_settings'
```

如果我们希望duplicate一个index，则可以获得一个index的meta，再通过index名称和－d参数，put进去。
```
curl -XPUT 'http://172.16.21.225:9200/e4_logstash-2014.10.19/' -d '
{"state":"open","settings":{"index.routing.allocation.include.DC":"e4","index.number_of_replicas":"1","index.version.created":"900199","index.number_of_shards":"2"},"mappings":{"B2B":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"E3-Internal-Hadoop":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Testing-Hadoop":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"JPFSys":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EDI_3PL_Biztalk":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"JSS_TEST":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Testing":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EDI_All_SOSJob":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Watchdog":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EDI_VF_Biztalk":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"NeweggCluster-06":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"NeweggCluster-05":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"NeweggCluster-04":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"MQService":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"NeweggCluster-02":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Mobile":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EC_Solr":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EIP_Front":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"NeweggFlash":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"E4-EC-Hadoop":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Website":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"cassandra_03":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"E3-EC-Hadoop":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"Windows":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"ECItemService":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"ActiveMQ":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"API-Notify":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}},"EDI_EaaS_MonitorLog":{"_ttl":{"enabled":true,"default":2592000000},"properties":{"LogTimeStamp":{"format":"dateOptionalTime","type":"date"}}}},"aliases":[]}
'
```

－－－－－－－END－－－－－－－－

伍涛@USA
