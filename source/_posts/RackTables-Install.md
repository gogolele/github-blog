title: RackTables Install
date: 2015-06-26 10:22:41
categories: dev
tags: RackTables
---

## Install LAMP Stack on CentOS 6

### httpd installation
```shell
[root@server]# yum install httpd -y
[root@server]# service httpd start
[root@server]# curl localhost:80
[root@server]# chkconfig httpd on
```

### mysqld installation
```shell
[root@server]# yum install mysql mysql-server -y
[root@server]# service mysqld start
[root@server]# chkconfig mysqld on
[root@server]# mysql_secure_installation
```

### php installation
```shell
[root@server]# yum install php -y
[root@server]# echo "test" | /var/www/html/testphp.php
[root@server]# service httpd start
[root@server]# curl http://localhost/testphp.php
[root@server]# yum install php-mysql -y
[root@server]# yum install phpmyadmin -y
[root@server]# vi /etc/httpd/conf.d/phpMyAdmin.conf
[...]
Alias /phpMyAdmin /usr/share/phpMyAdmin
Alias /phpmyadmin /usr/share/phpMyAdmin

#<Directory /usr/share/phpMyAdmin/>
#   <IfModule mod_authz_core.c>
#     # Apache 2.4
#     Require local
#   </IfModule>
#   <IfModule !mod_authz_core.c>
#     # Apache 2.2
#     Order Deny,Allow
#     Deny from All
#     Allow from 127.0.0.1
#     Allow from ::1
#   </IfModule>
#</Directory>
[...]

[root@server]# sudo find / -name "config.sample.inc.php"
[root@server]# cp /usr/share/doc/phpMyAdmin-4.0.10.10/examples/config.sample.inc.php /usr/share/phpMyAdmin/config.inc.php
[root@server]# vi /usr/share/phpMyAdmin/config.inc.php
[...]
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'http';
[...]

[root@server]# service httpd restart
```

## Install RackTables
```shell
[root@server]# yum install php-common php-cli php-ldap php-snmp php-pcntl
[root@server]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 50
Server version: 5.5.35 MySQL Community Server (GPL) by Remi

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database racktablesdb;
Query OK, 1 row affected (0.03 sec)

mysql> GRANT ALL ON racktablesdb.* TO racktablesuser@localhost IDENTIFIED by 'centos';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye

[root@server]# wget http://sourceforge.net/projects/racktables/files/RackTables-0.20.10.tar.gz
[root@server]# tar zxvf RackTables-0.20.10.tar.gz
[root@server]# cp -fr RackTables-0.20.6/wwwroot/ /var/www/html/racktables
[root@server]# chmod -R 777 /var/www/html/racktables/
[root@server]# touch /var/www/html/racktables/inc/secret.php
[root@server]# chown apache:apache /var/www/html/racktables/inc/secret.php; chmod 400 /var/www/html/racktables/inc/secret.php
[root@server]# service httpd restart
[root@server]# service mysqld restart
```

Navigate to http://ip-address/racktables or http://domain-name/racktables from your web browser and follow the onscreen instructions.

reference
http://www.unixmen.com/manage-data-center-rack-like-boss-racktables/
http://www.unixmen.com/install-lamp-server-in-centos-6-4-rhel-6-4/


-EOF-

TAO WU@CA, USA
