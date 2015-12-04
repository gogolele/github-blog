title: sed auto-configuration example
date: 2015-05-21 15:33:41
categories: dev
tags: [sed]
---

```shell
#!/bin/sh
#
# Wrote by Tao Wu
# 2015-05
hosts=( 'awsserver02' 'awsserver03' 'awsserver04' 'awsserver05' 'awsserver06' )
for m in ${hosts[@]};
do
#stop tsdmain
ssh $m "ps aux | grep -i TSDMain | grep -v grep | cut -c 9-15 | xargs kill -9"
#update config
ssh $m "sed -i 's/^\(tsd\.storage\.hbase\.zk_quorum\s*=\s*\).*$/\1aws-hbase11.mercury.corp,aws-hbase03.mercury.corp,aws-hbase01.mercury.corp,aws-hbase04.mercury.corp,aws-hbase02.mercury.corp/' /opt/app/opentsdb-2.0.1/opentsdb.conf"
#start
ssh $m "cd /opt/app/opentsdb-2.0.1/; ./start"
sleep 5
done
```

-EOF-
