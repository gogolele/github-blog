title: Use Sqoop to Transfer Data from SQLServer to Hive
date: 2015-04-10 20:03:47
categories: dev
tags: [hive, sqoop, hadoop]
---

## Preparation
hadoop-2.0.0-cdh4.3.0
hive-0.11
sqoop-1.4.3-cdh4.4.0

## Configuration
We should configure the Hadoop, and Hive in advance.

Go to the sqoop folder, open the conf/sqoop-env.sh, add HADOOP_HOME/HIVE_HOME, and HADOOP_USER_NAME if necessary.

```shell
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/opt/app/install/hadoop-current/share/hadoop/mapreduce1
#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/app/install/hadoop-current/share/hadoop/mapreduce1
#Set the path to where bin/hive is available
export HIVE_HOME=/opt/app/install/hive-current
export HADOOP_USER_NAME=hadoopuser
```

## Run Sqoop

1. create the hive table.

```shell
bin/sqoop create-hive-table --connect "jdbc:sqlserver://127.0.0.1;username=username;password=password;database=database" --table sqltable --hive-table hivetable
```

example
```shell
[ttt@e3qlog01 sqoop]$ bin/sqoop create-hive-table --connect "jdbc:sqlserver://127.0.0.1;username=username;password=password;database=database" --table sqltable --hive-table hivetable
Warning: /home/bigdata/hbase-0.92.1-cdh4.1.3 does not exist! HBase imports will fail.
Please set $HBASE_HOME to the root of your HBase installation.
Warning: /usr/lib/hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
15/04/10 15:41:46 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
15/04/10 15:41:46 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
15/04/10 15:41:46 WARN tool.BaseSqoopTool: It seems that you've specified at least one of following:
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --hive-home
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --hive-overwrite
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --create-hive-table
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --hive-table
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --hive-partition-key
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --hive-partition-value
15/04/10 15:41:46 WARN tool.BaseSqoopTool:      --map-column-hive
15/04/10 15:41:46 WARN tool.BaseSqoopTool: Without specifying parameter --hive-import. Please note that
15/04/10 15:41:46 WARN tool.BaseSqoopTool: those arguments will not be used in this session. Either
15/04/10 15:41:46 WARN tool.BaseSqoopTool: specify --hive-import to apply them correctly or remove them
15/04/10 15:41:46 WARN tool.BaseSqoopTool: from command line to remove this warning.
15/04/10 15:41:46 INFO tool.BaseSqoopTool: Please note that --hive-home, --hive-partition-key,
15/04/10 15:41:46 INFO tool.BaseSqoopTool:       hive-partition-value and --map-column-hive options are
15/04/10 15:41:46 INFO tool.BaseSqoopTool:       are also valid for HCatalog imports and exports
15/04/10 15:41:46 INFO manager.SqlManager: Using default fetchSize of 1000
15/04/10 15:41:47 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM [EC_CrawlerIPRulesWhiteList] AS t WHERE 1=
15/04/10 15:41:47 WARN hive.TableDefWriter: Column InDate had to be cast to a less precise type in Hive
15/04/10 15:41:47 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java cla
15/04/10 15:41:48 INFO hive.HiveImport: Loading uploaded data into Hive
15/04/10 15:41:48 INFO hive.HiveImport: /opt/app/install/hive-current/conf
15/04/10 15:41:48 INFO hive.HiveImport: /opt/app/install/hive-current
15/04/10 15:41:49 INFO hive.HiveImport: 15/04/10 15:41:49 WARN conf.HiveConf: DEPRECATED: Configuration property hive.metastortastore.uris if you are connecting to a remote metastore.
15/04/10 15:41:49 INFO hive.HiveImport:
15/04/10 15:41:49 INFO hive.HiveImport: Logging initialized using configuration in file:/opt/app/install/hive-0.11.0-e3interna
15/04/10 15:41:49 INFO hive.HiveImport: Hive history file=/tmp/ttt/hive_job_log_ttt_15174@server_201504101541_62
15/04/10 15:41:52 INFO hive.HiveImport: *******  add root tsk:Stage-0
15/04/10 15:41:52 INFO hive.HiveImport: *******  run new task : Stage-0
15/04/10 15:41:53 INFO hive.HiveImport: OK
15/04/10 15:41:53 INFO hive.HiveImport: Time taken: 3.581 seconds
15/04/10 15:41:53 INFO hive.HiveImport: Hive import complete.
```


2. import data from SQL Server to Hive

```shell
bin/sqoop import --connect "jdbc:sqlserver://127.0.0.1;username=username;password=password;database=database" --hive-import --table sqltable --hive-table hivetable
```

note: the string imported into Hive may contain blank striing, should use 'trim(field)' for hive sql.

-EOF-
