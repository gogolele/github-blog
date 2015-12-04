title: awk latency caused by buffer
date: 2015-08-20 21:17:51
tags: awk
categories: dev
---
# Awk latency in pipeline caused by buffer
when we us awk in pipeline processing, there will be latency as the buffer in awk.

the way to resolve this problem is to dissable the buffer.

use stdbuf -oL in centos 6.x
```shell
tail -F log.log |stdbuf -oL grep -i timecost | stdbuf -oL sed  's/</ /g' | stdbuf -oL sed 's/>/ /g' | stdbuf -oL sed 's/=/ /g' | stdbuf -oL awk '{print $2, $3, $9}' | stdbuf -oL awk '{dateStr = $1" "$2; cmd=("date +%s -d ""\""dateStr"\""); cmd | getline timestamp; close(cmd); printf "%d %d\n",timestamp,$3}' | stdbuf -oL awk -v host=`hostname -s` '{ print "put watservice.costTime " $1 " " $2 " host=" host}' | nc tsd-host tsd-port
```
this article explains very good on this issue.
what is the buffering? or why does my command line procedure no output? tail -f xx | grep xx | awk xx.
http://mywiki.wooledge.org/BashFAQ/009

-EOF-

TAO WU@CA, USA
