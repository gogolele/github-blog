title: Build Tez for CDH5.4.0
date: 2015-05-21 10:03:26
categories: dev
tags: [Tez, CDH5]
---

## checkout the Tez code
```shell
git clone https://github.com/apache/tez.git
git checkout tags/release-0.7.0
git checkout -b tristan
```

## modify pom.xml to use hadoop-2.6.0-cdh.5.4.0
```xml
<profile>
   <id>cdh5.4.0</id>
   <activation>
   <activeByDefault>false</activeByDefault>
   </activation>
   <properties>
     <hadoop.version>2.6.0-cdh5.4.0</hadoop.version>
   </properties>
   <pluginRepositories>
     <pluginRepository>
     <id>cloudera</id>
     <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
     </pluginRepository>
   </pluginRepositories>
   <repositories>
     <repository>
       <id>cloudera</id>
       <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
     </repository>
   </repositories>
</profile>
```

## mvn package
```shell
mvn -Pcdh5.4.0 clean package -Dtar -DskipTests=true -Dmaven.javadoc.skip=true
```

### method compile error
error message
```shell
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project tez-mapreduce: Compilation failure
[ERROR] /home/mil2048/Projects/tez/tez-mapreduce/src/main/java/org/apache/tez/mapreduce/hadoop/mapreduce/JobContextImpl.java:[57,8] org.apache.tez.mapreduce.hadoop.mapreduce.JobContextImpl is not abstract and does not override abstract method userClassesTakesPrecedence() in org.apache.hadoop.mapreduce.JobContext
```

add these code to resolve this problem,
```java
tez-mapreduce/src/main/java/org/apache/tez/mapreduce/hadoop/mapreduce/JobContextImpl.java
/**
 * Get the boolean value for the property that specifies which classpath
 * takes precedence when tasks are launched. True - user's classes takes
 * precedence. False - system's classes takes precedence.
 * @return true if user's classes should take precedence
 */
 @Override
public boolean userClassesTakesPrecedence() {
    return getJobConf().getBoolean(MRJobConfig.MAPREDUCE_JOB_USER_CLASSPATH_FIRST, false);
}
```
thanks to this article.
http://www.swiss-scalability.com/2015/05/hive-on-tez-on-cdh-love-everywhere.html



### protocol buffer problem
error message
```shell
org.apache.hadoop:hadoop-maven-plugins:2.6.0-cdh5.4.0:protoc (compile-protoc) on project tez-api
```

install the protocol buffer 2.5.0,
```shell
wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.bz2
tar xfvj protobuf-2.5.0.tar.bz2
cd protobuf-2.5.0/
sudo ./configure
sudo make
sudo make check
sudo make install
sudo ldconfig
protoc --version
```


### github connection problem
got connection problem to github.com. firewall block the git protocol, use http can resolve this problem.
error message
```shell
fatal: unable to connect to github.com:
github.com[0: 192.30.252.128]: errno=Connection timed out
```

resolve by this
```shell
git config --global url."https://".insteadOf git://
```

## Success

```shell
mvn -Pcdh5.4.0 clean package -Dtar -DskipTests=true -Dmaven.javadoc.skip=true

......

[INFO] Reactor Summary:
[INFO]
[INFO] tez ................................................ SUCCESS [  1.574 s]
[INFO] tez-api ............................................ SUCCESS [  6.522 s]
[INFO] tez-common ......................................... SUCCESS [  0.683 s]
[INFO] tez-runtime-internals .............................. SUCCESS [  1.136 s]
[INFO] tez-runtime-library ................................ SUCCESS [  2.585 s]
[INFO] tez-mapreduce ...................................... SUCCESS [  1.632 s]
[INFO] tez-examples ....................................... SUCCESS [  0.424 s]
[INFO] tez-dag ............................................ SUCCESS [  5.948 s]
[INFO] tez-tests .......................................... SUCCESS [  1.734 s]
[INFO] tez-ui ............................................. SUCCESS [ 15.040 s]
[INFO] tez-plugins ........................................ SUCCESS [  0.129 s]
[INFO] tez-yarn-timeline-history .......................... SUCCESS [  0.865 s]
[INFO] tez-yarn-timeline-history-with-acls ................ SUCCESS [  0.585 s]
[INFO] tez-mbeans-resource-calculator ..................... SUCCESS [  0.365 s]
[INFO] tez-tools .......................................... SUCCESS [  0.034 s]
[INFO] tez-dist ........................................... SUCCESS [ 39.968 s]
[INFO] Tez ................................................ SUCCESS [  0.033 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:19 min
[INFO] Finished at: 2015-05-20T16:30:30-07:00
[INFO] Final Memory: 63M/165M

```


we can use the tarball in the tes-dist/target/ folder.


-EOF-

TAO WU@CA, USA
