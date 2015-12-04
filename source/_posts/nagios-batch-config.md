title: nagios-batch-config
date: 2015-08-31 10:27:04
tags: [nagios, m4]
categories: dev
---

## Preparation

### install make
```shell
sudo yum install -y make
```
### expect to manage servers by group
```shell
tw79@T-Ubuntu:~/git-newegg/nagios/etc/dynamic-conf$ tree
.
├── hadoop-group
│   ├── hadoop.cfg
│   ├── hadoop.host.m4
│   ├── host-service-template.m4
│   └── Makefile
├── storm-group
│   ├── storm.cfg
│   ├── storm.host.m4
│   ├── host-service-template.m4
│   └── Makefile
```

### config a folder for nagios configuration
```cfg
# You can also tell Nagios to process all config files (with a .cfg
# extension) in a particular directory by using the cfg_dir
# directive as shown below:
cfg_dir=/usr/local/nagios/etc/dynamic-conf
```

## Nagios Batch Config

### write a MakeFile
```shell
clean:
        @echo "Removing config files..."
        rm -f *.cfg
%.cfg : %.host.m4
        m4 $< > $*.cfg
```
### write a template for nagios configuration
```shell
define(`GENER_CFG', `
  define hostescalation{
          host_name $1
          first_notification 2
          last_notification 0
          notification_interval 30
          contact_groups admins
  }

  define serviceescalation{
          host_name $1
          service_description openfiles,openport,cpu.use,memory.per,iostat.await,iostat.util
          first_notification 2
          last_notification 0
          notification_interval 30
          contact_groups admins
  }
  define host {
      use linux-server
  		host_name $1
      alias $2
      address $3
  		icon_image switch.gif
  		statusmap_image switch.gd2
  		2d_coords 100,200
      3d_coords 100,200,100
  }
  define service {
  		use local-service
      host_name $1
      check_command check_tsd!10000!30000!openfiles!host=$1
  		notification_interval 10
  		check_interval 6
      service_description openfiles
  		notifications_enabled 1
  }
  define service {
      use local-service
      host_name $1
      check_command check_tsd!1500!2500!openport!host=$1
      notification_interval 10
  		check_interval 6
      service_description openport
      notifications_enabled 1
  }


  define service {
      use local-service
      host_name $1
      check_command check_tsd!80!90!top.use!host=$1
      notification_interval 10
      check_interval 6
      service_description cpu.use
      notifications_enabled 1
   }


  define service {
      use local-service
      host_name $1
      check_command check_tsd!0.85!0.9!memory.per!host=$1
      notification_interval 10
      check_interval 6
      service_description memory.per
      notifications_enabled 1
  }

  define service {
      use local-service
      host_name $1
      check_command check_tsd!60!100!iostat.util!host=$1
      notification_interval 10
      check_interval 6
      service_description iostat.util
      notifications_enabled 1
  }

  define service {
      use local-service
      host_name $1
      check_command check_tsd!60!100!iostat.await!host=$1
      notification_interval 10
      check_interval 6
      service_description iostat.await
      notifications_enabled 1
   }
')
```

### define the server list on template
```shell
include(`host-service-template.m4')
GENER_CFG(`server21', `server21', `10.0.0.21')
GENER_CFG(`server22', `server22', `10.0.0.22')
GENER_CFG(`server23', `server23', `10.0.0.23')
GENER_CFG(`server24', `server24', `10.0.0.24')
GENER_CFG(`server25', `server25', `10.0.0.25')
GENER_CFG(`server26', `server26', `10.0.0.26')
GENER_CFG(`server27', `server27', `10.0.0.27')
GENER_CFG(`server28', `server28', `10.0.0.28')
GENER_CFG(`server29', `server29', `10.0.0.29')
GENER_CFG(`server30', `server30', `10.0.0.30')
GENER_CFG(`server31', `server31', `10.0.0.31')
GENER_CFG(`server32', `server32', `10.0.0.32')
GENER_CFG(`server33', `server33', `10.0.0.33')
GENER_CFG(`server34', `server34', `10.0.0.34')
GENER_CFG(`server35', `server35', `10.0.0.35')
GENER_CFG(`server36', `server36', `10.0.0.36')
```

### config example
```shell
$> make clean
$> make nagios.cfg
```

### restart nagios service
```shell
$> sudo service nagios restart
```
-EOF-

TAO WU@CA, USA
