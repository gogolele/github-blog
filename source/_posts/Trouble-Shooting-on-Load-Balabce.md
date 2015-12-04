title: Flume Agent Load Balance的一个问题
date: 2014-10-21 10:26:43
categories: dev
tags: [Strace, Flume]
---

周末发现flume的跨机房load balance好像有问题，本来流量是按照ip就近分配，从flume的tcp链接看到，
几乎所有的请求都集中到某一个机房，另一方机房没有流量。我进行了一下分析。

查看flume thrift端口61110的链接情况，其中一个机房链接达到680个，另一个只有27个，很明显，
两边的TCP链接差距很大。
```shell
   netstat -antup | grep 61110
```

进而我们查看ElasticSearch的数据情况，看起来正常的。但是机房A的index数据量达到2000万，而机房B的
index数据量只有70万。于是就有一个假设，B机房的数据通过机房A的agent进入数据存储端。
由于数据都带有机房标识，通过strace抓取机房A的agent进行确认。
```shell
  sudo yum install -y strace
  sudo strace -tt -s 10240 -f -o strace.out -e trace=all -p 22554
```
-f表示子线程的日志也进行trace。agent中有大量的线程，必须加入这个参数才可以trace。

在trace出来的文件中，我们很明显的看到，write方法中有很多其他机房的数据。这就证明了A机房的Agent
在处理B机房的数据。

为什么B机房的agent无法获取数据呢？原因是它在netscaler的状态显示为down。根本原因是因为B机房的
系统rebuild之后，之前存在的ip端口映射失效。即之前我们将8080端口通过iptables映射到80，这样
netscaler可以侦听状态。

于是我们修改iptables的端口映射
```shell
  sudo /sbin/chkconfig --list | grep iptables
  sudo yum install -y iptables
  sudo vim /etc/sysconfig/iptables-config
  # add rules
  sudo iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j REDIRECT
  --to-ports 8080
  # check rules
  sudo iptables -t nat -L -n
  # save rules
  sudo /etc/init.d/iptables save
  # restart iptables
  sudo /etc/init.d/iptables restart
```

这样就解决问题了。因为端口映射失效，造成load balance上的agent失效，造成traffic转移到另一个机房。


－－－－－－－END－－－－－－－－

伍涛@USA
