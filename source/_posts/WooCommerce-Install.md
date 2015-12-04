title: WooCommerce Install
date: 2015-07-20 09:35:17
categories: dev
tags: WooCommerce
---

OS: Ubuntu 14.04 LTS

## install lamp
remove mysql-server mysql-client first if necessary.
```shell
sudo -E apt-get remove mysql-server mysql-client
sudo -E apt-get purge mysql-core-5.5
sudo -E apt-get purge mysql-client-core-5.5
```
install lamp stack
```shell
sudo -E apt-get install apache2 mysql-server php5 libapache2-mod-php5 php5-gd php5-curl libssh2-php mysql-client php5-mysql
sudo vi /etc/php5/apache2/php.ini
  expose_php = Off
  allow_url_fopen = Off
sudo a2enmod rewrite
sudo service apache2 restart
```

## mysql script for wordpress
```sql
CREATE DATABASE wordpress;
CREATE USER wpadmin@localhost;
SET PASSWORD FOR wpadmin@localhost=PASSWORD("wpadmin");
GRANT ALL PRIVILEGES ON wordpress.* TO wpadmin@localhost IDENTIFIED BY 'wpadmin';
FLUSH PRIVILEGES;
```

## install wordpress
```shell
wget http://wordpress.org/latest.tar.gz
tar zxvf latest.tar.gz
cd wordpress/
sudo rsync -avz . /var/www/html/
sudo chown -R www-data:www-data /var/www/html/
sudo -u www-data cp wp-config-sample.php wp-config.php
sudo -u www-data vi wp-config.php
sudo vi /etc/apache2/sites-enabled/000-default.conf
sudo service apache2 restart
```

add this in /etc/apache2/sites-enabled/000-default.conf
```shell
ServerAdmin tonywutao@gmail.com
	DocumentRoot /var/www/html

    <Directory /var/www/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
```

set the proxy for wordpress if necessary. in /etc/www/html/wp-config.php
```
define('WP_PROXY_HOST', 'internal_proxy_host');
define('WP_PROXY_PORT', 'internal_proxy_port');
define('WP_PROXY_BYPASS_HOSTS', 'localhost');
```

## install woocommerce

reference
https://mos.meituan.com/library/25/how-to-install-wordpress-on-ubuntu/

-EOF-
