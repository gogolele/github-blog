title: 'Presto Installation'
date: 2015-07-30 16:51:51
categories: dev
tags: [Presto]
---

# Install Presto
## Preparation
jdk1.8_51
presto0.112
cdh5.4.0
hive1.1 meta server

## cluster planning
* server11, as coordinator
* server12 ~ server26, as worker

## ssh with no password
use ssh-keygen, and my fire-ssh.sh script

## configuration
* node.properties

```properties
node.id=2c4ba944-cc59-407a-8240-3cffb795ecff
node.environment=ecpresto
node.data-dir=/tmp/
```

* jvm.config, enable the jmx remote
```shell
-server
-Xmx16G
-Xms16G
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
-XX:+UseConcMarkSweepGC
-XX:+ExplicitGCInvokesConcurrent
-XX:+AggressiveOpts
-XX:+HeapDumpOnOutOfMemoryError
-XX:OnOutOfMemoryError=kill -9 %p
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9050
-Dcom.sun.management.jmxremote.local.only=false
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

* config.properties for coordinator
```properties
# sample nodeId to provide consistency across test runs
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8866
task.max-memory=1GB
discovery-server.enabled=true
discovery.uri=http://server11:8866

exchange.http-client.max-connections=1000
exchange.http-client.max-connections-per-server=1000
exchange.http-client.connect-timeout=1m
exchange.http-client.idle-timeout=1m

scheduler.http-client.max-connections=1000
scheduler.http-client.max-connections-per-server=1000
scheduler.http-client.connect-timeout=1m
scheduler.http-client.idle-timeout=1m

query.client.timeout=5m
query.max-age=30m

experimental-syntax-enabled=true
distributed-joins-enabled=true
```

* config.properties for worker
```properties
coordinator=false
http-server.http.port=8866
task.max-memory=1GB
discovery.uri=http://server11:8866
```

* hive config, etc/catalog/hive.properties
```properties
connector.name=hive-cdh5
hive.metastore.uri=thrift://hivemetaserver:9083
hive.config.resources=/home/domain/ttt/presto-work/presto-coordinator/etc/core-site.xml,/home/domain/ttt/presto-work/presto-coordinator/etc/hdfs-site.xml
```
## script for nodeid generation, start, stop, deployment

* node id generation, run-node-id.sh
use uuidgen to generate uuid for each node
```shell
a=$(uuidgen);sed -i 's/^\(node\.id\s*=\s*\).*$/\1'$(echo $a)'/' etc/node.properties
```

* start script, run.sh
```shell
PATH=/home/domain/ttt/presto-work/java-current/bin:$PATH bin/launcher start
```

* stop script, stop-presto.sh
```shell
ps aux | grep -i presto | grep -v airpal | grep -v grep | cut -c 9-15 | xargs kill -9
```

* deploy script, deploy-presto.sh
```shell
#!/bin/sh
hosts=( 'server12' 'server13' 'server14' 'server15' 'server16' 'server17' 'server18' 'server19' 'server20' 'server21' 'server22' 'server23' 'server24' 'server25' 'server26')
for m in ${hosts[@]};
do
## mkdir presto-work
ssh $m "mkdir ~/presto-work;"
## copy and install java
scp /home/domain/ttt/presto-work/jdk*.tar.gz $m:/home/domain/ttt/presto-work/
ssh $m "cd ~/presto-work; tar zxvf jdk*.tar.gz; ln -s /home/domain/ttt/presto-work/jdk1.8.0_51 java-current"
## copy and install presto
scp /home/domain/ttt/presto-work/presto-worker.tar.gz $m:/home/domain/ttt/presto-work/
ssh $m "cd ~/presto-work; tar zxvf presto-worker.tar.gz;cd presto-worker; source /etc/profile;./run-node-id.sh;./run.sh"
sleep 1
done
```

## run client
download the presto client jar
```shell
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.112/presto-cli-0.112-executable.jar
mv presto-cli-0.112-executable.jar presto-cli
```

run-client.sh
```shell
PATH=/home/domain/ttt/presto-work/java-current/bin:$PATH java -Xms8g -Xmx8g -jar /home/domain/ttt/presto-work/presto-cli --server server11:8866 --catalog hive --schema mars
```


## check cluster status
```shell
presto:mars> select * from system.runtime.nodes;
               node_id                |         http_uri          |   node_version    | coordinator | active
--------------------------------------+---------------------------+-------------------+-------------+--------
 2c4ba944-cc59-407a-8240-3cffb795ecff | http://10.0.0.41:8866 | presto-main:0.112 | true        | true
 dc76d898-8525-45d7-9987-8ebe9092fca0 | http://10.0.0.45:8866 | presto-main:0.112 | false       | true
 21b80c88-bea2-4f8c-b1c1-b5139db92a27 | http://10.0.0.56:8866 | presto-main:0.112 | false       | true
 92251a97-390f-4449-a7b9-a453f4e4fdda | http://10.0.0.51:8866 | presto-main:0.112 | false       | true
 de0ee2bd-4064-49a5-b033-04436822955b | http://10.0.0.55:8866 | presto-main:0.112 | false       | true
 ce1ccf2f-9cb6-4489-8ed2-cac880eae491 | http://10.0.0.49:8866 | presto-main:0.112 | false       | true
 8f0a52ba-6f58-4e95-99d5-480a4ce420d0 | http://10.0.0.50:8866 | presto-main:0.112 | false       | true
 a2f1ed5f-dbd2-46d5-b699-82b7752cf3f7 | http://10.0.0.44:8866 | presto-main:0.112 | false       | true
 7df814c4-021b-4c60-9569-1753eb1f9d24 | http://10.0.0.52:8866 | presto-main:0.112 | false       | true
 10a79203-dc12-4b10-862a-656b3939ed45 | http://10.0.0.53:8866 | presto-main:0.112 | false       | true
 65e37afc-e439-4e44-b28b-92ff9248a1af | http://10.0.0.42:8866 | presto-main:0.112 | false       | true
 0d8ebb4b-20ea-459a-bc74-8d46194503d4 | http://10.0.0.48:8866 | presto-main:0.112 | false       | true
 07e4cdee-24ce-424d-939e-7bbf7ee7498b | http://10.0.0.54:8866 | presto-main:0.112 | false       | true
 c336f28d-c02b-4366-a411-9bcfaf745553 | http://10.0.0.47:8866 | presto-main:0.112 | false       | true
 ba64ce7b-4850-4aff-a446-3a79cc603bd0 | http://10.0.0.46:8866 | presto-main:0.112 | false       | true
 31330ed9-bba2-4836-957e-bcde0cedf965 | http://10.0.0.43:8866 | presto-main:0.112 | false       | true
```

## run sql query
```shell
presto:mars> select parent, count(1) as childnum from (select a.key as child, b.key as parent from base_table a inner join base_table b on a.parent=b.key ) c group by parent order by childnum desc limit 50;
     parent     | childnum
----------------+----------
                | 26807897
 0C0-0024-00001 |      417
 0C2-0029-00007 |      178
 81-644-001     |      117
 0E5-000H-00020 |       78
 76-998-409     |       50
 72-104-048     |       50
 81-103-036     |       46
 58-108-574     |       45
 0C9-00CK-00003 |       44
 72-104-053     |       44
 76-998-410     |       42
 76-998-422     |       42
 9ATBBVR00H8571 |       42
 00E-003P-00002 |       42
 76-101-436     |       42
 76-998-416     |       42
 0C9-00CK-00005 |       41
 58-108-642     |       41
 12-120-302     |       41
 76-998-421     |       41
 9ATBBVR00A0209 |       40
 75-973-066     |       40
 9ATBBVR0094522 |       40

Query 20150801_152116_00016_ugftv, FINISHED, 16 nodes
Splits: 717 total, 717 done (100.00%)
1:33 [149M rows, 32.4GB] [1.61M rows/s, 357MB/s]
```

# trouble shooting

## version not match in config.properties
```shell
presto.version=testversion  ## in coordinator, copy from dev config
presto-version=0.112        ## not config, this is default value,
```

the workers cannot connect to coordinator if version mismatch

```shell
presto:mars> select * from system.runtime.nodes;
               node_id                |         http_uri          |   node_version    | coordinator | active
--------------------------------------+---------------------------+-------------------+-------------+--------
 2c4ba944-cc59-407a-8240-3cffb795ecff | http://10.0.0.41:8866 | testversion       | true        | true
 dc76d898-8525-45d7-9987-8ebe9092fca0 | http://10.0.0.45:8866 | presto-main:0.112 | false       | false  
 21b80c88-bea2-4f8c-b1c1-b5139db92a27 | http://10.0.0.56:8866 | presto-main:0.112 | false       | false  
 92251a97-390f-4449-a7b9-a453f4e4fdda | http://10.0.0.51:8866 | presto-main:0.112 | false       | false  
 de0ee2bd-4064-49a5-b033-04436822955b | http://10.0.0.55:8866 | presto-main:0.112 | false       | false  
 ce1ccf2f-9cb6-4489-8ed2-cac880eae491 | http://10.0.0.49:8866 | presto-main:0.112 | false       | false  
 8f0a52ba-6f58-4e95-99d5-480a4ce420d0 | http://10.0.0.50:8866 | presto-main:0.112 | false       | false  
 a2f1ed5f-dbd2-46d5-b699-82b7752cf3f7 | http://10.0.0.44:8866 | presto-main:0.112 | false       | false  

 7df814c4-021b-4c60-9569-1753eb1f9d24 | http://10.0.0.52:8866 | presto-main:0.112 | false       | false  
 10a79203-dc12-4b10-862a-656b3939ed45 | http://10.0.0.53:8866 | presto-main:0.112 | false       | false  
 65e37afc-e439-4e44-b28b-92ff9248a1af | http://10.0.0.42:8866 | presto-main:0.112 | false       | false  
 0d8ebb4b-20ea-459a-bc74-8d46194503d4 | http://10.0.0.48:8866 | presto-main:0.112 | false       | false  
 07e4cdee-24ce-424d-939e-7bbf7ee7498b | http://10.0.0.54:8866 | presto-main:0.112 | false       | false  
 c336f28d-c02b-4366-a411-9bcfaf745553 | http://10.0.0.47:8866 | presto-main:0.112 | false       | false  
 ba64ce7b-4850-4aff-a446-3a79cc603bd0 | http://10.0.0.46:8866 | presto-main:0.112 | false       | false  
 31330ed9-bba2-4836-957e-bcde0cedf965 | http://10.0.0.43:8866 | presto-main:0.112 | false       | false

```
issue on github
https://github.com/facebook/presto/issues/3382

## Task exceed 1GB memory
should increase the task memrory configuration. in config.properties
```shell
task.max-memory=4GB
```

## presto job will hang when large cdv hive table
fixed by larger the task max-memory
```shell
task.max-memory=4GB
```

issue on github
https://github.com/facebook/presto/issues/3390

-EOF-

Tao WU@CA,USA
