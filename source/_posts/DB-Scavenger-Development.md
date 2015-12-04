title: DB Scavenger Development
date: 2015-03-26 14:47:15
categories: dev
tags: [sqlserver,crontab]
---

For deleting expired data in SQL Server timely, I wrote simple program to do this task.

## Write JavaCode

Scavenger.java
```java
package com.tonywutao.scavenger;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.List;


public class Scavenger {
  public static void main(String[] args) {
    Constants.init();
    runScavenger();
  }

  public static List<DBSentence> sentenceLoad(){
    List<DBSentence> lstDBSentence = new ArrayList<DBSentence>();
    DBSentence currentDBSentence = null;

    String [] arrUser = Constants.DB_USERNAME.split(",");
    String [] arrPass = Constants.DB_PASSWORD.split(",");
    String [] arrUrl = Constants.DB_URL.split(",");
    String [] arrTable = Constants.DB_TABLE.split(",");
    String [] arrDateColumn = Constants.DB_DATE_COLUMN.split(",");
    String [] arrDateExpire = Constants.DB_DATE_EXPIRE.split(",");
    int count = arrUser.length;

    if(arrPass.length != count || arrUrl.length != count || arrTable.length != count || arrDateColumn.length != count || arrDateExpire.length != count){
      Util.printLog("configuration pairs are not in same length");
    }

    for (int i=0; i<count; i++){
      currentDBSentence = makeDBSentence(arrUser[i], arrPass[i], arrUrl[i], arrTable[i], arrDateColumn[i], arrDateExpire[i]);
      lstDBSentence.add(currentDBSentence);
    }
    return lstDBSentence;
  }

  private static DBSentence makeDBSentence(String username, String password, String dbUrl, String dbTable, String dateColumn, String dateExpire){
    String expireDateString = getDateString(dateExpire);
    String sql = " delete top(" +Constants.BATCH_SIZE+ ")from " + dbTable + " where " + dateColumn  + " < '" + expireDateString + "'";
    DBSentence dbs = new DBSentence();
    dbs.setUrl(dbUrl);
    dbs.setUsername(username);
    dbs.setPassword(password);
    dbs.setSql(sql);
    return dbs;
  }

  private static String getDateString(String dateRange){
    int num = Integer.parseInt(dateRange.substring(0,dateRange.length()-1));
    String dateString = null;
    if(dateRange.endsWith("d")){
      Date now = new Date(System.currentTimeMillis());
      Calendar c = Calendar.getInstance();
      c.setTime(now);
      c.add(Calendar.DATE, -num);
      Date expire = (Date) c.getTime();
      dateString = Util.date2String(expire);
      } else if (dateRange.endsWith("h")){
        Date now = new Date(System.currentTimeMillis());
        Calendar c = Calendar.getInstance();
        c.setTime(now);
        c.add(Calendar.HOUR, -num);
        Date expire = (Date) c.getTime();
        dateString = Util.date2String(expire);
        } else if (dateRange.endsWith("m")){
          Date now = new Date(System.currentTimeMillis());
          Calendar c = Calendar.getInstance();
          c.setTime(now);
          c.add(Calendar.MONTH, -num);
          Date expire = (Date) c.getTime();
          dateString = Util.date2String(expire);
          } else if (dateRange.endsWith("f")){
            Date now = new Date(System.currentTimeMillis());
            Calendar c = Calendar.getInstance();
            c.setTime(now);
            c.add(Calendar.MONTH, -num);
            Date expire = (Date) c.getTime();
            dateString = Util.date2String(expire);
          }

          return dateString;
        }

        public static void runOneSentence(DBSentence dbs){
          boolean bContinue = true;
          int loopCount = 0;
          while (bContinue){
            bContinue = runOneBatchSentence(dbs);
            loopCount ++;
            Util.printLog(" delete " +loopCount + " batches successfullly, total = " + (loopCount*Constants.BATCH_SIZE));
            try {
              Thread.currentThread().sleep(50L);
              } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
              }
            }
          }

          public static boolean runOneBatchSentence(DBSentence dbs) {
            boolean bContinue = false;;
            try {
              Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
              } catch (Exception e) {
                System.out.println("class.forname error: ");
                e.printStackTrace();
              }
              Connection con = null;
              Statement st = null;
              ResultSet rs = null;
              try {
                con = DriverManager.getConnection(dbs.getUrl(), dbs.getUsername(), dbs.getPassword());
                st = con.createStatement();
                int result = st.executeUpdate(dbs.getSql());
                bContinue = result >= Constants.BATCH_SIZE ? true : false;
                } catch (Exception e) {
                  e.printStackTrace();
                  Util.printLog("sql="+dbs.getSql()+","+e.getMessage());
                  } finally {
                    if (rs != null) {
                      try {
                        rs.close();
                        } catch (SQLException e) {
                          e.printStackTrace();
                        }
                      }
                      if (st != null) {
                        try {
                          st.close();
                          } catch (SQLException e) {
                            e.printStackTrace();
                          }
                        }
                        if (con != null) {
                          try {
                            con.close();
                            } catch (SQLException e) {
                              e.printStackTrace();
                            }
                          }
                        }
                        return bContinue;
                      }

                      public static void runScavenger(){
                        Util.printLog(" start scavenger... ");
                        List<DBSentence> lstDbs = sentenceLoad();
                        if(lstDbs == null) return;
                        Util.printLog(" loaded db sentence size = "+ lstDbs.size());
                        int loopCount = 0;
                        for(DBSentence dbs: lstDbs){
                          runOneSentence(dbs);
                          loopCount ++;
                          Util.printLog(" db sentence [" +loopCount+ "]  is completed= "+ dbs.toString());
                        }
                      }
                    }

```

Constants.java
```java
package com.tonywutao.scavenger;

import java.io.IOException;
import java.util.Properties;

/**
* public constants definition
*
* @author tw79
*
*/
public class Constants {
  protected static String DB_USERNAME = "XXX";
  protected static String DB_PASSWORD = "XXX";
  protected static String DB_URL = "jdbc:sqlserver://XXX_SQL;databaseName=XXX";
  protected static String DB_TABLE = "XXX.dbo.XXX";
  protected static String DB_DATE_COLUMN = "InDate";
  protected static String DB_DATE_EXPIRE = "5d";
  protected static int BATCH_SIZE = 2000;

  public static void init(){
    try {
      Properties props = new Properties();
      props.load(Constants.class.getClassLoader().getResourceAsStream(
      "scavenger.properties"));
      DB_USERNAME = props.getProperty("db.username", DB_USERNAME);
      DB_PASSWORD = props.getProperty("db.password", DB_PASSWORD);
      DB_URL = props.getProperty("db.url", DB_URL);
      DB_TABLE = props.getProperty("db.table", DB_TABLE);
      DB_DATE_COLUMN = props.getProperty("db.date.column", DB_DATE_COLUMN);
      DB_DATE_EXPIRE = props.getProperty("db.date.expire", DB_DATE_EXPIRE);

      Util.printLog("=========== configuration =================");
      Util.printLog("DB_USERNAME =" + DB_USERNAME);
      Util.printLog("DB_PASSWORD =" + DB_PASSWORD);
      Util.printLog("DB_URL =" + DB_URL);
      Util.printLog("DB_TABLE =" + DB_TABLE);
      Util.printLog("DB_DATE_COLUMN =" + DB_DATE_COLUMN);
      Util.printLog("DB_DATE_EXPIRE =" + DB_DATE_EXPIRE);
      Util.printLog("===========================================");
    } catch (IOException e) {
      Util.printLog("load configuration file error");
      e.printStackTrace();
    }
  }
}

```

pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.tonywutao.scavenger</groupId>
<artifactId>scavenger</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>scavenger</name>
<description>scavenger for db log</description>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-dependency-plugin</artifactId>
<executions>
<execution>
<id>copy-dependencies</id>
<phase>package</phase>
<goals>
<goal>copy-dependencies</goal>
</goals>
<configuration>
<outputDirectory>${project.build.directory}/lib</outputDirectory>
<overWriteReleases>false</overWriteReleases>
<overWriteSnapshots>false</overWriteSnapshots>
<overWriteIfNewer>true</overWriteIfNewer>
<excludeArtifactIds>junit,serlvet-api</excludeArtifactIds>
</configuration>
</execution>
</executions>

</plugin>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-resources-plugin</artifactId>
<version>2.4</version>
<executions>
<execution>
<id>copy-resources</id>
<phase>package</phase>
<goals>
<goal>copy-resources</goal>
</goals>
<configuration>
<encoding>UTF-8</encoding>
<outputDirectory>${project.build.directory}</outputDirectory><!-- copy resource files -->
<resources>
<resource>
<directory>src/main/resources/</directory>
<includes>
<include>ipdata.properties</include>
<include>log4j.properties</include>
<include>start-program.sh</include>
<include>stop-program.sh</include>
</includes>
<filtering>true</filtering>  <!--这个为true可以进行变量替换，替换配置文件中的${***} -->
</resource>
</resources>
</configuration>
</execution>
</executions>
</plugin>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-shade-plugin</artifactId>
<version>2.0</version>
<configuration>
<shadedArtifactAttached>true</shadedArtifactAttached>
</configuration>
<executions>
<execution>
<phase>package</phase>
<goals>
<goal>shade</goal>
</goals>
<configuration>
<filters>
<filter>
<artifact>*:*</artifact>
<excludes>
<exclude>META-INF/*.SF</exclude>
<exclude>META-INF/*.DSA</exclude>
<exclude>META-INF/*.RSA</exclude>
</excludes>
</filter>
</filters>
</configuration>
</execution>
</executions>
</plugin>
</plugins>
</build>
<dependencies>
<dependency>
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.17</version>
</dependency>
<dependency>
<groupId>commons-logging</groupId>
<artifactId>commons-logging</artifactId>
<version>1.1.3</version>
</dependency>
<dependency>
<groupId>com.microsoft.sqlserver</groupId>
<artifactId>sqljdbc4</artifactId>
<version>4.0</version>
</dependency>
</dependencies>
</project>
```

## Maven Compile
I use maven-shade-plugin to build the jar with dependency libraries.

```shell
mvn clean package
```

the first time we encountered a problem

```
Exception in thread "main" java.lang.SecurityException: Invalid signature file digest for Manifest main attributes
```

To resolve the problem, use this exclude configuration in pom.xml

```xml
configuration>
<filters>
  <filter>
    <artifact>*:*</artifact>
    <excludes>
      <exclude>META-INF/*.SF</exclude>
      <exclude>META-INF/*.DSA</exclude>
      <exclude>META-INF/*.RSA</exclude>
    </excludes>
  </filter>
</filters>
</configuration>
```

## Crontab Configuration

start-program.sh

```shell
#!/bin/bash
java -Xmx512m -Xms512m -cp ".:/home/ecbigdata/db-scavenger/scavenger-0.0.1-SNAPSHOT-shaded.jar" com.tonywutao.scavenger.Scavenger

```

stop-program.sh

```shell
#!/bin/bash
ps aux | grep -i scavenger | grep -v grep | cut -c 9-15 | xargs kill -9
```

modify the /etc/crontab for timer job, add one line

method 1,  redirect console log to file in start-program.sh

```shell
#!/bin/bash
#nohup /opt/install/java-current/bin/java -Xmx512m -Xms512m -cp ".:/home/ecbigdata/db-scavenger/scavenger-0.0.1-SNAPSHOT-shaded.jar" com.tonywutao.scavenger.Scavenger  >> /home/ecbigdata/db-scavenger/scavenger.log 2 >&1 < /dev/null &
```

```shell
* */6 * * * ecbigdata sh /home/ecbigdata/db-scavenger/start-program.sh
```

method 2, redict console log to file in /etc/crontab

```shell
#!/bin/bash
/opt/install/java-current/bin/java -Xmx512m -Xms512m -cp ".:/home/ecbigdata/db-scavenger/scavenger-0.0.1-SNAPSHOT-shaded.jar" com.tonywutao.scavenger.Scavenger

```

```shell
* */6 * * * ecbigdata sh /home/ecbigdata/db-scavenger/start-program.sh >> /home/ecbigdata/db-scavenger/crontab.log
```

if I want to find the log containing a specific string in current folder, GREP is the tool

```shell
grep -nr "XXX" *
```

-EOF-

Tao Wu @ CA, USA
