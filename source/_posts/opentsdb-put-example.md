title: opentsdb put example
date: 2015-08-20 21:29:27
tags: opentsdb
categories: dev
---

## Client Example (Push Metrics)

### Telnet Example
log exmaple
```
<INFO><2015-08-20 00:00:24> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch$RowKeyUpdateThread.run(HBaseCDH430RegionWatch.java:237)refresh rowkeylist success
<INFO><2015-08-20 00:00:25> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch.watch(HBaseCDH430RegionWatch.java:158)watch task success , timeCost=1463
<INFO><2015-08-20 00:00:28> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch.watch(HBaseCDH430RegionWatch.java:123)watch start,table name = IM_ItemBase
<INFO><2015-08-20 00:00:30> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch.watch(HBaseCDH430RegionWatch.java:158)watch task success , timeCost=1520
<INFO><2015-08-20 00:00:33> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch.watch(HBaseCDH430RegionWatch.java:123)watch start,table name = IM_ItemBase
<INFO><2015-08-20 00:00:33> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch.watch(HBaseCDH430RegionWatch.java:158)watch task success , timeCost=626
<INFO><2015-08-20 00:00:34> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch$RowKeyUpdateThread.run(HBaseCDH430RegionWatch.java:213)refresh rowkeylist task start...
<INFO><2015-08-20 00:00:34> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch$RowKeyUpdateThread$1.call(HBaseCDH430RegionWatch.java:219)start to refresh
<INFO><2015-08-20 00:00:34> newegg.ec.warden.watch.hbase.HBaseCDH430RegionWatch$RowKeyUpdateThread.run(HBaseCDH430RegionWatch.java:237)refresh rowkeylist success
```

```shell
tail -F log.log |stdbuf -oL grep -i timecost | stdbuf -oL sed  's/</ /g' | stdbuf -oL sed 's/>/ /g' | stdbuf -oL sed 's/=/ /g' | stdbuf -oL awk '{print $2, $3, $9}' | stdbuf -oL awk '{dateStr = $1" "$2; cmd=("date +%s -d ""\""dateStr"\""); cmd | getline timestamp; close(cmd); printf "%d %d\n",timestamp,$3}' | stdbuf -oL awk -v host=`hostname -s` '{ print "put watservice.costTime " $1 " " $2 " host=" host}' | nc tsd-server 4242
```
```shell
tail -F log.log |stdbuf -oL grep -i timecost | stdbuf -oL sed  's/</ /g' | stdbuf -oL sed 's/>/ /g' | stdbuf -oL sed 's/=/ /g' | stdbuf -oL awk '{print $2, $3, $9}' | stdbuf -oL awk '{dateStr = $1" "$2; cmd=("date +%s -d ""\""dateStr"\""); cmd | getline timestamp; close(cmd); printf "%d %d\n",timestamp,1}' | stdbuf -oL awk -v host=`hostname -s` '{ print "put watservice.requestCount " $1 " " $2 " host=" host}' | nc tsd-server 4242  
```

note:
*  tail -F : to avoid file (log.log) move to another file
*  stdbuf -oL : to disable the pipeline buffer for awk / sed, buffer can cause the latency.
*  nc: NetCat, use this command to send data to TCP / UDP

data format
```shell
put metrics-name timestamp value tag=value
```

data example
```shell
put watservice.costTime 1440095418 1438 host=server10
put watservice.costTime 1440095423 1449 host=server10
put watservice.costTime 1440095427 1461 host=server10
put watservice.costTime 1440095432 1454 host=server10
put watservice.costTime 1440095436 1453 host=server10
put watservice.costTime 1440095440 627 host=server10
put watservice.costTime 1440095444 1516 host=server10
```
Note: We suggest to use second-level unix timestamp, not millsecond-level unix timestamp

### HTTP Example

post this message to opentsdb server
```shell
http://tsd-server:4242/api/put
```

```json
{
    "metric": "watservice.costTime",
    "timestamp": 1440106203,
    "value": 4000,
    "tags": {
       "host": "server10"
    }
}
```

Curl example
```shell
curl -H "Content-Type: application/json" -X POST -d '{
    "metric": "watservice.costTime",
    "timestamp": 1440106203,
    "value": 4000,
    "tags": {
       "host": "server10"
    }
}
' http://tsd-server:4242/api/put
```

## Reference
the relative open-source project
http://grafana.org/
http://opentsdb.net/
https://www.nagios.org

### 1 OpenTSDB HTTP API to put metrics
http://opentsdb.net/docs/build/html/api_http/put.html

### 2 OpenTSDB Telnet API to put metrics
The easiest way to get started with OpenTSDB is to open up a terminal or telnet client, connect to your TSD and issue a put command and hit 'enter'. If you are writing a program, simply open a socket, print the string command with a new line and send the packet. The telnet command format is:
put <metric> <timestamp> <value> <tagk1=tagv1[ tagk2=tagv2 ...tagkN=tagvN]>
For example:

  put sys.cpu.user 1356998400 42.5 host=webserver01 cpu=0

Each put can only send a single data point. Don't forget the newline character, e.g. \n at the end of your command.

-EOF-

TAO WU@CA,USA
