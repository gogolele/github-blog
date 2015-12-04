title: "Deploy ElasticSearch Cluster "
date: 2015-04-10 17:22:32
categories: dev
tags: elasticsearch
---

## Preparation
jdk1.7.0_40.tar
elasticsearch-1.4.4.tar.gz
head.zip
bigdesk.tar

## ssh-pass

first, we should install expect for automatically breaking ssh.

```shell
sudo yum -y install expect
```

use the ssh-pass.sh to break through the ssh login.

```shell
#!/bin/bash
#
# Copyright (c) Finnbarr P. Murphy 2006
# modified by Tao Wu, 2014
#
# ---- start configurables ----
PATH=/usr/sbin:/usr/bin:/sbin:/bin
# ----- end configurables -----
function usage {
  printf "Usage: setupkeys -U remote-username -P remote-password -H remote-host -u local-username\n"
  exit 2
}
# --- script starts here
echo
(( $# == 0 )) && usage
username=""
password=""
host=""
localuser=""
while getopts "u:P:U:H:" OPTION
do
case $OPTION in
U) username="$OPTARG";;
P) password="$OPTARG";;
H) host="$OPTARG";;
u) localuser="$OPTARG";;
esac
done
# --- basic argument checking
if [[ -z "$username" ]]; then
echo "ERROR - No username entered."
exit 1
fi
if [[ -z "$password" ]]; then
echo "ERROR - No passed entered."
exit 1
fi
if [[ -z "$host" ]]; then
echo "ERROR - No host entered."
exit 1
fi
if [[ -z "$localuser" ]]; then
echo "ERROR - No localuser entered."
exit 1
fi
echo -n "Checking connectivity with $host. "
/bin/ping -q -c 2 $host > /dev/null 2>&1
RESULT=$?
if (( RESULT == 1 )); then
echo; echo "ERROR - could not ping $host."
exit 1
fi
echo "System is alive."
# --- check for $localuser public and private ssh keys
# --- we need to be $localuser here when using ssh-* utilities
echo -n "Checking for $localuser ssh key files. "
SSH_KEYS_FOUND=0
if [[ -d $HOME ]]; then
if [[ -s $HOME/.ssh/id_rsa && -s $HOME/.ssh/id_rsa.pub ]]; then
/usr/bin/ssh-keygen -e -f $HOME/.ssh/id_rsa.pub | grep "2048-bit RSA" > /dev/null 2>&1
RESULT=$?
if (( RESULT == 0 )); then
SSH_KEYS_FOUND=1
fi
fi
fi
echo "Not found"
rm -rf $HOME/.ssh
mkdir $HOME/.ssh
chmod 700 $HOME/.ssh
chown -R $localuser:$localuser $HOME/.ssh
/usr/bin/ssh-keygen -q -t rsa -N "" -f $HOME/.ssh/id_rsa
echo "New ssh key files generated (RSA protocol)"
fi
# --- add $localname's public key to $username@$host authorized keys
TMPUSER=expectscript-user.$$
cat <<EOT > $TMPUSER
#!/usr/bin/expect
if {[llength \$argv] != 4} {
  puts "usage: \$argv0 localuser username password host"
  exit 1
}
log_file -a expectscript-user.log
log_user 0
set localuser [lindex \$argv 0]
set username [lindex \$argv 1]
set password [lindex \$argv 2]
set host [lindex \$argv 3]
set timeout 60
spawn /usr/bin/ssh-copy-id -i \$localuser/.ssh/id_rsa.pub \$username@\$host
expect {
  "assword: " {
    send "$password\n"
    expect {
      "again." { exit 1 }
      "expecting." { }
      timeout { exit 1 }
    }
  }
  "(yes/no)? " {
    send "yes\n"
    expect {
      "assword: " {
        send "$password\n"
        expect {
          "again." { exit 1 }
          "expecting." { }
          timeout { exit 1 }
        }
      }
    }
  }
}
exit 0
EOT
echo -n "Copying $localuser's public key to $host. "
chmod 755 $TMPUSER
sleep 3
./$TMPUSER $HOME $username $password $host
RESULT=$?
rm -f $TMPUSER
if (( RESULT == 0 )); then
echo "Succeeded"
rm -f expectscript-user.log
else
echo "Failed"
echo "ERROR: Check expectscript-user.log"
exit 1
fi
# we are done
echo "Setup completed. Goodbye."
exit 0
```

write a shell scripts for this cluster. fire-ssh.sh

```shell
#!/bin/sh
#
# Wrote by Tristan Wu
# 2015-04

user=ttt
pass='******'

hosts=('server101' 'server102' 'server103' 'server104' 'server105' 'server106' 'server107' 'server108' 'server109' 'cas110')
for m in ${hosts[@]};
do
./ssh-pass.sh -U $user -P $pass -H $m -u $user
sleep 1
done

```

run this scripts:

```shell
[ttt@server101 ~]$ ./fire-pass.sh
Checking connectivity with server101. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server101. Succeeded
Setup completed. Goodbye.
Checking connectivity with server102. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server102. Succeeded
Setup completed. Goodbye.
Checking connectivity with server103. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server103. Succeeded
Setup completed. Goodbye.
Checking connectivity with server104. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server104. Succeeded
Setup completed. Goodbye.
Checking connectivity with server105. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server105. Succeeded
Setup completed. Goodbye.
Checking connectivity with server106. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server106. Succeeded
Setup completed. Goodbye.
Checking connectivity with server107. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server107. Succeeded
Setup completed. Goodbye.
Checking connectivity with server108. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server108. Succeeded
Setup completed. Goodbye.
Checking connectivity with server109. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to server109. Succeeded
Setup completed. Goodbye.
Checking connectivity with cas110. System is alive.
Checking for ttt ssh key files. Found
Copying ttt's public key to cas110. Succeeded
Setup completed. Goodbye.
```

## head / bigdesk install for operation

try plugin install provided by elasticsearch, we encountered problem

```shell
[ttt@server101 es-current]$ bin/plugin -install mobz/elasticsearch-head
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/elasticsearch/plugins/PluginManager : Unsupported major.minor version 51.0
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClassCond(ClassLoader.java:631)
at java.lang.ClassLoader.defineClass(ClassLoader.java:615)
at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
Could not find the main class: org.elasticsearch.plugins.PluginManager.  Program will exit.

```

the reason is the current java version is 1.6, but the elasticsearch is compiled by jdk 1.7, should use the new version, and config in elasticsearch start script.

```shell
[ttt@server101 es-current]$ java -version
java version "1.6.0_43"
Java(TM) SE Runtime Environment (build 1.6.0_43-b01)
Java HotSpot(TM) 64-Bit Server VM (build 20.14-b01, mixed mode)

[ttt@server101 ~]$ tar xvf jdk1.7.0_40.tar
[ttt@server101 ~]$ ln -s /home/CASSANDRA/ttt/jdk1.7.0_40 java-current
[ttt@server101 es-current]$ vi bin/elasticsearch
# Maven will replace the project.name with elasticsearch below. If that
# hasn't been done, we assume that this is not a packaged version and the
# user has forgotten to run Maven to create a package.

JAVA_HOME=/home/CASSANDRA/ttt/java-current

IS_PACKAGED_VERSION='elasticsearch'
if [ "$IS_PACKAGED_VERSION" != "elasticsearch" ]; then
cat >&2 << EOF
Error: You must build the project with Maven or download a pre-built package
before you can run Elasticsearch. See 'Building from Source' in README.textile
or visit http://www.elasticsearch.org/download to get a pre-built package.
EOF
exit 1
fi
```

Try again, encountered the network problem for download head/bigdesk, so change to download mannually, and deploy them to es-current/plugin/ folder,
then start the elasticsearch, it works.

```shell
[ttt@server101 es-current]$ bin/plugin -install mobz/elasticsearch-head
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip...
Failed to install mobz/elasticsearch-head, reason: failed to download out of all possible locations..., use --verbose to get detailed information
[ttt@server101 es-current]$ bin/plugin -install mobz/elasticsearch-head
-> Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip...
Failed to install mobz/elasticsearch-head, reason: failed to download out of all possible locations..., use --verbose to get detailed information
[ttt@server101 es-current]$ wget https://github.com/mobz/elasticsearch-head/archive/master.zip
--2015-04-09 16:07:53--  https://github.com/mobz/elasticsearch-head/archive/master.zip
Connecting to 172.16.22.102:3129... connected.
Proxy request sent, awaiting response... 302 Found
Location: https://codeload.github.com/mobz/elasticsearch-head/zip/master [following]
--2015-04-09 16:07:54--  https://codeload.github.com/mobz/elasticsearch-head/zip/master
Connecting to 172.16.22.102:3129... connected.
Proxy request sent, awaiting response... 200 OK
Length: 894178 (873K) [application/zip]
Saving to: `master'

100%[====================================================================================================================================================================>] 894,178     1.75M/s   in 0.5s

2015-04-09 16:07:55 (1.75 MB/s) - `master' saved [894178/894178]

[ttt@server101 es-current]$ mv master master.zip
[ttt@server101 es-current]$ mv master.zip plugins/
[ttt@server101 es-current]$ cd plugins/
[ttt@server101 plugins]$ ls
master.zip
[ttt@server101 plugins]$ unzip master.zip
[ttt@server101 plugins]$ cp -r master/* head/_site/
[ttt@server101 plugins]$ wget https://github.com/lukas-vlcek/bigdesk/tarball/master
--2015-04-09 16:18:08--  https://github.com/lukas-vlcek/bigdesk/tarball/master
Connecting to 172.16.22.102:3129... connected.
Proxy request sent, awaiting response... 302 Found
Location: https://codeload.github.com/lukas-vlcek/bigdesk/legacy.tar.gz/master [following]
--2015-04-09 16:18:09--  https://codeload.github.com/lukas-vlcek/bigdesk/legacy.tar.gz/master
Connecting to 172.16.22.102:3129... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [application/x-gzip]
Saving to: `master'

[  <=>                                                                                                                                                                ] 311,629     1.11M/s   in 0.3s

2015-04-09 16:18:10 (1.11 MB/s) - `master' saved [311629]
[ttt@server101 plugins]$ mv master bigdesk.zip
[ttt@server101 plugins]$ tar xvf bigdesk.zip
[ttt@server101 bigdesk]$ mv lukas-vlcek-bigdesk-505b32e _site
```

## start one node for testing

start elasticsearch successfully

```shell
[ttt@server101 es-current]$ bin/elasticsearch
[2015-04-09 16:47:44,274][INFO ][node                     ] [server101.CASSANDRA.corp] version[1.4.4], pid[30384], build[c88f77f/2015-02-19T13:05:36Z]
[2015-04-09 16:47:44,275][INFO ][node                     ] [server101.CASSANDRA.corp] initializing ...
[2015-04-09 16:47:44,294][INFO ][plugins                  ] [server101.CASSANDRA.corp] loaded [], sites [head, bigdesk]
[2015-04-09 16:47:50,516][INFO ][node                     ] [server101.CASSANDRA.corp] initialized
[2015-04-09 16:47:50,517][INFO ][node                     ] [server101.CASSANDRA.corp] starting ...
[2015-04-09 16:47:50,781][INFO ][transport                ] [server101.CASSANDRA.corp] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/10.0.0.101:9300]}
[2015-04-09 16:47:50,802][INFO ][discovery                ] [server101.CASSANDRA.corp] es-ec/cCRReh4RSVmTGhrOzln_0A
[2015-04-09 16:47:53,862][INFO ][cluster.service          ] [server101.CASSANDRA.corp] new_master [server101.CASSANDRA.corp][cCRReh4RSVmTGhrOzln_0A][server101.CASSANDRA.corp][inet[/10.0.0.101:9300]], reason: zen-disco-join (elected_as_master)
[2015-04-09 16:47:53,902][INFO ][http                     ] [server101.CASSANDRA.corp] bound_address {inet[/0:0:0:0:0:0:0:0:9200]}, publish_address {inet[/10.0.0.101:9200]}
[2015-04-09 16:47:53,903][INFO ][node                     ] [server101.CASSANDRA.corp] started
[2015-04-09 16:48:26,185][INFO ][node                     ] [server101.CASSANDRA.corp] stopping ...
[2015-04-09 16:48:26,289][INFO ][node                     ] [server101.CASSANDRA.corp] stopped
[2015-04-09 16:48:26,289][INFO ][node                     ] [server101.CASSANDRA.corp] closing ...
[2015-04-09 16:48:26,311][INFO ][node                     ] [server101.CASSANDRA.corp] closed

```

## deploy to cluster

then write the script to remote deploy elasticsearch in other nodes, we encounter this problem. the HOSTNAME environmental viriable can not be recognized by Remote SSH. We should add " source
/etc/profile " before.

```shell
{1.4.4}: Setup Failed ...
- IllegalArgumentException[Could not resolve placeholder 'HOSTNAME']
java.lang.IllegalArgumentException: Could not resolve placeholder 'HOSTNAME'
at org.elasticsearch.common.property.PropertyPlaceholder.parseStringValue(PropertyPlaceholder.java:124)
at org.elasticsearch.common.property.PropertyPlaceholder.replacePlaceholders(PropertyPlaceholder.java:81)
at org.elasticsearch.common.settings.ImmutableSettings$Builder.replacePropertyPlaceholders(ImmutableSettings.java:1060)
at org.elasticsearch.node.internal.InternalSettingsPreparer.prepareSettings(InternalSettingsPreparer.java:101)
at org.elasticsearch.bootstrap.Bootstrap.initialSettings(Bootstrap.java:106)
at org.elasticsearch.bootstrap.Bootstrap.main(Bootstrap.java:177)
at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:32)

```

The remote deploy shell scripts:

```shell
#!/bin/sh
#
# Wrote by Tao Wu
# 2015-04

hosts=('server102' 'server103' 'server104' 'server105' 'server106' 'server107' 'server108' 'server109' 'cas110')
for m in ${hosts[@]};
do
scp ~/stop-elasticsearch.sh $m:~/
ssh $m "cd ~/ ; rm -rf elasticsearch-1.4.4* ;rm -rf jdk1.7.0_40*; rm -rf java-current; rm -rf es-current"
scp ~/jdk*.tar $m:~/
scp ~/elasticsearch-1.4.4-deploy.tar.gz $m:~/
ssh $m "cd ~/; ./stop-elasticsearch.sh"
ssh $m "cd ~/;tar xvf jdk*.tar; ln -s /home/CASSANDRA/ttt/jdk1.7.0_40 java-current;tar zxvf elasticsearch-1.4.4-deploy.tar.gz; ln -s /home/CASSANDRA/ttt/elasticsearch-1.4.4 es-current; source /etc/profile ; cd es-current;bin/elasticsearch -d -Xmx8g -Xms8g"
#    ssh $m "cd ~/;tar xvf jdk*.tar; ln -s /home/CASSANDRA/ttt/jdk1.7.0_40 java-current; tar zxvf elasticsearch-1.4.4-deploy.tar.gz; ln -s /home/CASSANDRA/ttt/elasticsearch-1.4.4 /home/CASSANDRA/ttt/es-current"
sleep 1
done
```

## elasticsearch configuration file

```lang

##################### Elasticsearch Configuration Example #####################

# This file contains an overview of various configuration settings,
# targeted at operations staff. Application developers should
# consult the guide at <http://elasticsearch.org/guide>.
#
# The installation procedure is covered at
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/setup.html>.
#
# Elasticsearch comes with reasonable defaults for most settings,
# so you can try it out without bothering with configuration.
#
# Most of the time, these defaults are just fine for running a production
# cluster. If you're fine-tuning your cluster, or wondering about the
# effect of certain configuration option, please _do ask_ on the
# mailing list or IRC channel [http://elasticsearch.org/community].

# Any element in the configuration can be replaced with environment variables
# by placing them in ${...} notation. For example:
#
#node.rack: ${RACK_ENV_VAR}

# For information on supported formats and syntax for the config file, see
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/setup-configuration.html>


################################### Cluster ###################################

# Cluster name identifies your cluster for auto-discovery. If you're running
# multiple clusters on the same network, make sure you're using unique names.
#
cluster.name: es-ec


#################################### Node #####################################

# Node names are generated dynamically on startup, so you're relieved
# from configuring them manually. You can tie this node to a specific name:
#
node.name: ${HOSTNAME}

# Every node can be configured to allow or deny being eligible as the master,
# and to allow or deny to store the data.
#
# Allow this node to be eligible as a master node (enabled by default):
#
#node.master: true
#
# Allow this node to store data (enabled by default):
#
#node.data: true

# You can exploit these settings to design advanced cluster topologies.
#
# 1. You want this node to never become a master node, only to hold data.
#    This will be the "workhorse" of your cluster.
#
#node.master: false
#node.data: true
#
# 2. You want this node to only serve as a master: to not store any data and
#    to have free resources. This will be the "coordinator" of your cluster.
#
#node.master: true
#node.data: false
#
# 3. You want this node to be neither master nor data node, but
#    to act as a "search load balancer" (fetching data from nodes,
#    aggregating results, etc.)
#
#node.master: false
#node.data: false

# Use the Cluster Health API [http://localhost:9300/_cluster/health], the
# Node Info API [http://localhost:9300/_nodes] or GUI tools
# such as <http://www.elasticsearch.org/overview/marvel/>,
# <http://github.com/karmi/elasticsearch-paramedic>,
# <http://github.com/lukas-vlcek/bigdesk> and
# <http://mobz.github.com/elasticsearch-head> to inspect the cluster state.

# A node can have generic attributes associated with it, which can later be used
# for customized shard allocation filtering, or allocation awareness. An attribute
# is a simple key value pair, similar to node.key: value, here is an example:
#
#node.rack: rack314

# By default, multiple nodes are allowed to start from the same installation location
# to disable it, set the following:
#node.max_local_storage_nodes: 1


#################################### Index ####################################

# You can set a number of options (such as shard/replica options, mapping
# or analyzer definitions, translog settings, ...) for indices globally,
# in this file.
#
# Note, that it makes more sense to configure index settings specifically for
# a certain index, either when creating it or by using the index templates API.
#
# See <http://elasticsearch.org/guide/en/elasticsearch/reference/current/index-modules.html> and
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/indices-create-index.html>
# for more information.

# Set the number of shards (splits) of an index (5 by default):
#
index.number_of_shards: 5

# Set the number of replicas (additional copies) of an index (1 by default):
#
index.number_of_replicas: 1

# Note, that for development on a local machine, with small indices, it usually
# makes sense to "disable" the distributed features:
#
#index.number_of_shards: 1
#index.number_of_replicas: 0

# These settings directly affect the performance of index and search operations
# in your cluster. Assuming you have enough machines to hold shards and
# replicas, the rule of thumb is:
#
# 1. Having more *shards* enhances the _indexing_ performance and allows to
#    _distribute_ a big index across machines.
# 2. Having more *replicas* enhances the _search_ performance and improves the
#    cluster _availability_.
#
# The "number_of_shards" is a one-time setting for an index.
#
# The "number_of_replicas" can be increased or decreased anytime,
# by using the Index Update Settings API.
#
# Elasticsearch takes care about load balancing, relocating, gathering the
# results from nodes, etc. Experiment with different settings to fine-tune
# your setup.

# Use the Index Status API (<http://localhost:9300/A/_status>) to inspect
# the index status.


#################################### Paths ####################################

# Path to directory containing configuration (this file and logging.yml):
#
#path.conf: /path/to/conf

# Path to directory where to store index data allocated for this node.
#
#path.data: /path/to/data
#
# Can optionally include more than one location, causing data to be striped across
# the locations (a la RAID 0) on a file level, favouring locations with most free
# space on creation. For example:
#
#path.data: /path/to/data1,/path/to/data2

# Path to temporary files:
#
#path.work: /path/to/work

# Path to log files:
#
#path.logs: /path/to/logs

# Path to where plugins are installed:
#
#path.plugins: /path/to/plugins


#################################### Plugin ###################################

# If a plugin listed here is not installed for current node, the node will not start.
#
#plugin.mandatory: mapper-attachments,lang-groovy


################################### Memory ####################################

# Elasticsearch performs poorly when JVM starts swapping: you should ensure that
# it _never_ swaps.
#
# Set this property to true to lock the memory:
#
#bootstrap.mlockall: true

# Make sure that the ES_MIN_MEM and ES_MAX_MEM environment variables are set
# to the same value, and that the machine has enough memory to allocate
# for Elasticsearch, leaving enough memory for the operating system itself.
#
# You should also make sure that the Elasticsearch process is allowed to lock
# the memory, eg. by using `ulimit -l unlimited`.


############################## Network And HTTP ###############################

# Elasticsearch, by default, binds itself to the 0.0.0.0 address, and listens
# on port [9300-9300] for HTTP traffic and on port [9300-9400] for node-to-node
# communication. (the range means that if the port is busy, it will automatically
# try the next port).

# Set the bind address specifically (IPv4 or IPv6):
#
#network.bind_host: 192.168.0.1

# Set the address other nodes will use to communicate with this node. If not
# set, it is automatically derived. It must point to an actual IP address.
#
#network.publish_host: 192.168.0.1

# Set both 'bind_host' and 'publish_host':
#
#network.host: 192.168.0.1

# Set a custom port for the node to node communication (9300 by default):
#
#transport.tcp.port: 9300

# Enable compression for all communication between nodes (disabled by default):
#
#transport.tcp.compress: true

# Set a custom port to listen for HTTP traffic:
#
http.port: 9200

# Set a custom allowed content length:
#
#http.max_content_length: 100mb

# Disable HTTP completely:
#
#http.enabled: false


################################### Gateway ###################################

# The gateway allows for persisting the cluster state between full cluster
# restarts. Every change to the state (such as adding an index) will be stored
# in the gateway, and when the cluster starts up for the first time,
# it will read its state from the gateway.

# There are several types of gateway implementations. For more information, see
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-gateway.html>.

# The default gateway type is the "local" gateway (recommended):
#
gateway.type: local

# Settings below control how and when to start the initial recovery process on
# a full cluster restart (to reuse as much local data as possible when using shared
# gateway).

# Allow recovery process after N nodes in a cluster are up:
#
gateway.recover_after_nodes: 10

# Set the timeout to initiate the recovery process, once the N nodes
# from previous setting are up (accepts time value):
#
#gateway.recover_after_time: 5m

# Set how many nodes are expected in this cluster. Once these N nodes
# are up (and recover_after_nodes is met), begin recovery process immediately
# (without waiting for recover_after_time to expire):
#
gateway.expected_nodes: 10


############################# Recovery Throttling #############################

# These settings allow to control the process of shards allocation between
# nodes during initial recovery, replica allocation, rebalancing,
# or when adding and removing nodes.

# Set the number of concurrent recoveries happening on a node:
#
# 1. During the initial recovery
#
#cluster.routing.allocation.node_initial_primaries_recoveries: 4
#
# 2. During adding/removing nodes, rebalancing, etc
#
#cluster.routing.allocation.node_concurrent_recoveries: 2

# Set to throttle throughput when recovering (eg. 100mb, by default 20mb):
#
#indices.recovery.max_bytes_per_sec: 20mb

# Set to limit the number of open concurrent streams when
# recovering a shard from a peer:
#
#indices.recovery.concurrent_streams: 5


################################## Discovery ##################################

# Discovery infrastructure ensures nodes can be found within a cluster
# and master node is elected. Multicast discovery is the default.

# Set to ensure a node sees N other master eligible nodes to be considered
# operational within the cluster. This should be set to a quorum/majority of
# the master-eligible nodes in the cluster.
#
#discovery.zen.minimum_master_nodes: 1

# Set the time to wait for ping responses from other nodes when discovering.
# Set this option to a higher value on a slow or congested network
# to minimize discovery failures:
#
#discovery.zen.ping.timeout: 3s

# For more information, see
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-discovery-zen.html>

# Unicast discovery allows to explicitly control which nodes will be used
# to discover the cluster. It can be used when multicast is not present,
# or to restrict the cluster communication-wise.
#
# 1. Disable multicast discovery (enabled by default):
#
discovery.zen.ping.multicast.enabled: false
#
# 2. Configure an initial list of master nodes in the cluster
#    to perform discovery when new nodes (master or data) are started:
#
discovery.zen.ping.unicast.hosts: ["server101:9300", "server102:9300","server103:9300","server104:9300","server105:9300","server106:9300","server107:9300","server108:9300","server109:9300","cas110:9300"]

# EC2 discovery allows to use AWS EC2 API in order to perform discovery.
#
# You have to install the cloud-aws plugin for enabling the EC2 discovery.
#
# For more information, see
# <http://elasticsearch.org/guide/en/elasticsearch/reference/current/modules-discovery-ec2.html>
#
# See <http://elasticsearch.org/tutorials/elasticsearch-on-ec2/>
# for a step-by-step tutorial.

# GCE discovery allows to use Google Compute Engine API in order to perform discovery.
#
# You have to install the cloud-gce plugin for enabling the GCE discovery.
#
# For more information, see <https://github.com/elasticsearch/elasticsearch-cloud-gce>.

# Azure discovery allows to use Azure API in order to perform discovery.
#
# You have to install the cloud-azure plugin for enabling the Azure discovery.
#
# For more information, see <https://github.com/elasticsearch/elasticsearch-cloud-azure>.

################################## Slow Log ##################################

# Shard level query and fetch threshold logging.

#index.search.slowlog.threshold.query.warn: 10s
#index.search.slowlog.threshold.query.info: 5s
#index.search.slowlog.threshold.query.debug: 2s
#index.search.slowlog.threshold.query.trace: 500ms

#index.search.slowlog.threshold.fetch.warn: 1s
#index.search.slowlog.threshold.fetch.info: 800ms
#index.search.slowlog.threshold.fetch.debug: 500ms
#index.search.slowlog.threshold.fetch.trace: 200ms

#index.indexing.slowlog.threshold.index.warn: 10s
#index.indexing.slowlog.threshold.index.info: 5s
#index.indexing.slowlog.threshold.index.debug: 2s
#index.indexing.slowlog.threshold.index.trace: 500ms

################################## GC Logging ################################

#monitor.jvm.gc.young.warn: 1000ms
#monitor.jvm.gc.young.info: 700ms
#monitor.jvm.gc.young.debug: 400ms

#monitor.jvm.gc.old.warn: 10s
#monitor.jvm.gc.old.info: 5s
#monitor.jvm.gc.old.debug: 2s

################################## Security ################################

# Uncomment if you want to enable JSONP as a valid return transport on the
# http server. With this enabled, it may pose a security risk, so disabling
# it unless you need it is recommended (it is disabled by default).
#
#http.jsonp.enable: true

```

-EOF-

TAO WU@CA, USA
