title: Deploy CDH5.3.3 by Cloudera Manager
date: 2015-04-18 21:23:41
categories: dev
tags: [Hadoop, ClouderaManager]
---

## Preparation
1. Choose the latest Cloudera version.
I choose the lasted 5.3.3, which is released in 04/09/2015.

2. CentOS 5.8
```shell
[ttt@server101 hadoop-work]$ cat /etc/redhat-release
CentOS release 5.8 (Final)
```

3. Cloudera Manager 5.3.3
So we should download this cloudera manager
http://archive.cloudera.com/cm5/cm/5/cloudera-manager-el5-cm5.3.3_x86_64.tar.gz

4. JDK1.7_67
I used the JDK1.6_43, HBase encountered the connection problem.
As the recommendation from cloudear, we should use the JDK1.7_67 or JDK1.8_11

5. Cloudera Hadoop Installation Parcel.
Online download the hadoop installation parcel will cost a long time, and in our company network, we can not download files from cloudera, so I change to download it first, and take advantage of the offline parcel mode. I downloaded the parcel from this url:
DH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel (1.5GB)
http://archive.cloudera.com/cdh5/parcels/latest/CDH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel
DH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel.sha1
http://archive.cloudera.com/cdh5/parcels/latest/CDH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel.sha1
manifest.json
http://archive.cloudera.com/cdh5/parcels/latest/manifest.json

6. User Account
I cannot use the root user, I use a domain account with sudo rights, I encountered many permission problems during the install process.
user account: scm-user

## Installation
### planning
Machine|Role
:---------------|:---------------
server101|Hadoop NameNode,HBase Master,Zookeeper,CM-Server,CM-Agent
server102|Hadoop NameNode Standby,HBase Master Standby,Zookeeper,DataNode,RegionServer,CM-Agent
server103|Hadoop NameNoe,HBase Master,Zookeeper,DataNode,RegionServer,CM-Agent
server104|DataNode,RegionServer,CM-Agent
server105|DataNode,RegionServer,CM-Agent
server106|DataNode,RegionServer,CM-Agent
server107|DataNode,RegionServer,CM-Agent
server108|DataNode,RegionServer,CM-Agent
server109|DataNode,RegionServer,CM-Agent
e4cas110|DataNode,RegionServer,CM-Agent

### check the network configuration
MTU=1500

### stop the iptables service.
```shell
[ttt@server101 hadoop-work]$ sudo service iptables stop
```

### ssh pass (on server101 and server102)
```shell
sudo yum install -y expect
@server101 hadoop-work]$ vi fire.sh
user=ttt
pass='XXXX'
hosts=('server101' 'server102' 'server103' 'server104' 'server105' 'server106' 'server107' 'server108' 'server109' 'e4cas110')
for m in ${hosts[@]};
do
./ssh-pass.sh -U $user -P $pass -H $m -u $user
sleep 1
done
```

### initialize the DATABASE in mysql
I use another existed mysql instance for Cloudera Manager.
At first, should grant previlges to a user for cloudera manager:
```shell
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '888888' WITH GRANT OPTION;
```
Still encountered the problem that clouder manager cannot access the database "cm". So I created a new account named "hadoop", and for scm in cloudera manager, create a "scm" user with password "scm".
```shell
mysql> GRANT ALL PRIVILEGES ON *.* TO 'hadoop'@'%' IDENTIFIED BY '888888' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'scm'@'%' IDENTIFIED BY '888888' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

after that, use the schema initializing script, and ran successfully:
```shell
[ttt@server101 hadoop-work]$  cd ~/hadoop-work
[ttt@server101 hadoop-work]$  cm-5.3.3/share/cmf/schema/scm_prepare_database.sh mysql cm -h localhost -u hadoop -p --scm-host localhost scm scm scm
```


### configure the scm-agent
The scm--agent configuration file in cm-5.3.3/
1. configure the server_host, which refers to scm-server hosts.
2. configure the data file without permission problems.
3. configure the customize clouder parcels path.
```shell
[ttt@server101 hadoop-work]$ vi cm-5.3.3/etc/cloudera-scm-agent/config.ini
[ttt@server101 hadoop-work]$ cat cm-5.3.3/etc/cloudera-scm-agent/config.ini | head
[General]
# Hostname of the CM server.
server_host=server101

# Port that the CM server is listening on.
server_port=7182

## It should not normally be necessary to modify these.
# Port that the CM agent should listen on.
# listening_port=9000

...

lib_dir=/data/hadoop/lib/cloudera-scm-agent
parcel_dir=/data/hadoop/cloudera/parcels
...
```
### ssh scm-agent to all the nodes.

### start scm-agent and scm-server
```shell
cm-5.3.3/etc/init.d/cloudera-scm-agent start
cm-5.3.3/etc/init.d/cloudera-scm-server start
```
the cloudera-scm-server should waiting 5 - 10 minutes to start, when successfully started, you should see the 7180 port started for cloudera manager. Use admin/admin to login the cloudera manager portal.

### configure via cloudera manager
1. choose the free version, already no 50 node limits in cm-5.3.3
2. put the downloaded cloudera parcels into the specific folder "/data/hadoop/cloudera/parcels", not, you should change the "sha1" file to "sha". after this, you do not need to download installing parcels from internet. Clouder manager will let you to select the downloaded parcels.
should list the files like this
```shell
[ttt@server101 hadoop-work]$ ls -al /data/hadoop/cloudera/parcels
total 1527208
drwxr-xr-x  3 ttt domain users       4096 Apr 18 00:19 .
drwxr-xr-x  6 ttt domain users       4096 Apr 17 11:29 ..
-rw-r--r--  1 ttt domain users 1562266343 Apr 17 06:50 CDH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel
-rw-r--r--  1 ttt domain users         41 Apr 17 06:50 CDH-5.3.3-1.cdh5.3.3.p0.5-el5.parcel.sha
-rw-r--r--  1 ttt domain users      42475 Apr 17 06:50 manifest.json
```
3. distributed & activation.
waiting for about 10 minutes to activation. when done, we can select instances to install.

## Problems
### permission problems.
we encountered permission problems many times. list here:
```shell
hosts=('server101' 'server102' 'server103' 'server104' 'server105' 'server106' 'server107' 'server108' 'server109' 'e4cas110')
for m in ${hosts[@]};
do
scp ~/jdk1.7.0_67-cloudera.tar.gz $m:~/
scp ~/hadoop-work/cm-5.3.3/etc/default/cloudera-scm-agent $m:~/hadoop-work/cm-5.3.3/etc/default/cloudera-scm-agent
ssh $m "sudo cp ~/jdk1.7.0_67-cloudera.tar.gz /usr/java; cd /usr/java; sudo tar zxvf jdk1.7.0_67-cloudera.tar.gz"
ssh $m "cd /home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/init.d; source /etc/profile; ./cloudera-scm-agent restart"

scp /home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/init.d/cloudera-scm-agent $m:/home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/init.d/
ssh $m "sudo mkdir /var/run/cloudera-scm-agent;sudo chown ttt /var/run/cloudera-scm-agent; sudo chgrp 'domain users' /var/run/cloudera-scm-agent"
 ssh $m "cd /home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/init.d; source /etc/profile; ./cloudera-scm-agent hard_restart_confirmed"
ssh $m "source /etc/profile; ps aux | grep -i cm-5.3.3 | grep -v grep | cut -c 9-15 | xargs kill -9;"
ssh $m "sudo mv /etc/security/limits.d/ttt.conf /etc/security/limits.d/90-ttt.conf"
sleep 1
ssh $m "sudo mkdir /var/log/hadoop-hdfs; sudo chown -R ttt /var/log/hadoop-hdfs; sudo chgrp -R  'domain users' /var/log/hadoop-hdfs"
ssh $m "sudo mkdir /var/log/hadoop-yarn; sudo chown -R ttt /var/log/hadoop-yarn; sudo chgrp -R  'domain users' /var/log/hadoop-yarn"
ssh $m "sudo mkdir /var/lib/hadoop-yarn; sudo chown -R ttt /var/lib/hadoop-yarn; sudo chgrp -R  'domain users' /var/lib/hadoop-yarn"
ssh $m "sudo mkdir /var/log/cloudera-scm-eventserver; sudo chown -R ttt /var/log/cloudera-scm-eventserver; sudo chgrp -R  'domain users' /var/log/cloudera-scm-eventserver"
ssh $m "sudo mkdir /var/lib/cloudera-scm-eventserver; sudo chown -R ttt /var/lib/cloudera-scm-eventserver; sudo chgrp -R  'domain users' /var/lib/cloudera-scm-eventserver"
ssh $m "sudo mkdir /var/log/cloudera-scm-alertpublisher; sudo chown -R ttt /var/log/cloudera-scm-alertpublisher; sudo chgrp -R  'domain users' /var/log/cloudera-scm-alertpublisher "
ssh $m "sudo mkdir /var/log/hbase; sudo chown -r ttt /var/log/hbase; sudo chgrp -r  'domain users' /var/log/hbase"
ssh $m "sudo mkdir /var/log/zookeeper; sudo chown -r ttt /var/log/zookeeper; sudo chgrp -r  'domain users' /var/log/zookeeper"
ssh $m "sudo mkdir /var/run/hdfs-sockets; sudo chown -R ttt /var/run/hdfs-sockets; sudo chgrp -R  'domain users' /var/run/hdfs-sockets"
ssh $m " sudo chgrp -R  'domain users' /var/run/hdfs-sockets"
scp  ~/hadoop-install/ttt.conf $m:~/hadoop-work/ttt.conf
ssh $m "sudo cp ~/hadoop-work/ttt.conf /etc/security/limits.d/ttt.conf; sudo chown ttt /etc/security/limits.d/ttt.conf; sudo chgrp -R  'domain users' /etc/security/limits.d/ttt.conf"
ssh $m "rm -rf /opt/cloudera/parcels;"
scp /home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/cloudera-scm-agent/config.ini $m:/home/MERCURY/ttt/hadoop-work/cm-5.3.3/etc/cloudera-scm-agent
done

```
### max locked memory limit problem
prolem like this, will stop the data node starting.
```shell
java.lang.RuntimeException: Cannot start datanode because the configured max locked memory size (dfs.datanode.max.locked.memory) of 4294967296 bytes is more than the datanode's available RLIMIT_MEMLOCK ulimit of 32768 bytes.
at org.apache.hadoop.hdfs.server.datanode.DataNode.startDataNode(DataNode.java:1054)
at org.apache.hadoop.hdfs.server.datanode.DataNode.<init>(DataNode.java:411)
at org.apache.hadoop.hdfs.server.datanode.DataNode.makeInstance(DataNode.java:2301)
at org.apache.hadoop.hdfs.server.datanode.DataNode.instantiateDataNode(DataNode.java:2188)
at org.apache.hadoop.hdfs.server.datanode.DataNode.createDataNode(DataNode.java:2235)
at org.apache.hadoop.hdfs.server.datanode.DataNode.secureMain(DataNode.java:2411)
at org.apache.hadoop.hdfs.server.datanode.DataNode.main(DataNode.java:2435)
```
configure the a ulimit file to /etc/security/limit.d/ to change configuration.
```shell
[ttt@server101 hadoop-work]$ ls -al /etc/security/limits.d
total 24
drwxr-xr-x 2 root root         4096 Apr 17 09:29 .
drwxr-xr-x 5 root root         4096 Nov 20 16:16 ..
-rw-r--r-- 1 dd11 domain users  208 Jul  3  2013 90-cassandra.conf
-rw-r--r-- 1 ttt domain users  152 Apr 17 08:05 90-ttt.conf
[ttt@server101 hadoop-work]$ cat /etc/security/limits.d/90-ttt.conf
ttt soft nofile 32768
ttt soft nproc 65536
ttt hard nofile 1048576
ttt hard nproc unlimited
ttt hard memlock unlimited
ttt soft memlock unlimited
[ttt@server101 hadoop-work]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 2102144
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 32768
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65536
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```
This change can be work after ssh re-connection. In our installation, we should restart the scm-agent and scm-server to make sure this configuration works.

### HBase master can not work, timeout for each query
after HBase cluter started, we found that the master hang for every query, and got 60000 ms timeout every time. it was resolved by upgrading the java to JDK1.7_67.
This problem is resolved by these configurations:
1. Open /etc/default/cloudera-scm-agent.
2. Set the CMF_AGENT_JAVA_HOME environment variable to the java home in your environment. For example, you might modify the file to include the following line:
3. export CMF_AGENT_JAVA_HOME=/usr/custom_java
4. Save and close the cloudera-scm-agent file.
5. Restart the Cloudera Manager Agent using the following command:
6. sudo service cloudera-scm-agent restart

-EOF-

Tao WU@CA, USA
