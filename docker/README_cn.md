## 首先编译最外层
```bash
mvn clean install  -DskipTests -Dmaven.test.skip=true -Dscala-2.12 -Dspark3.4  -T4C --settings ~/.m2/settings-flink.xml
```
这样可以得到spark3.4的版本
```bash
cd docker/
./build_local_docker_images.sh #编译基本镜像，这个脚本有问题不编译镜像
#编译镜像可以采用
cd hoodie/hadoop/
mvn clean pre-integration-test -DskipTests -Ddocker.compose.skip=true -Ddocker.build.skip=false
```
在code的docker插件下可以看到编译的docker正在增加
[]: # Path: README_cn.md



Hadoop3.0开始默认端口的更改
Namenode 端口:
https服务的端口50470 --> 9871
NameNode web管理端口50070 --> 9870配置文件hdfs-site.xml
namenode RPC交互端口,用于获取文件系统metadata信息。8020 --> 9820配置文件core-site.xml
Secondary NN 端口:
暂未了解到50091 --> 9869
secondary NameNode web管理端口50090 --> 9868
Datanode 端口:
datanode的IPC服务器地址和端口50020 --> 9867配置文件hdfs-site.xml
datanode控制端口,用于数据传输50010 --> 9866配置文件hdfs-site.xml
https服务的端口50475 --> 9865
datanode的HTTP服务器和端口50075 --> 9864配置文件hdfs-site.xml



hudi-hadoop-hive-docker
mvn clean pre-integration-test -DskipTests -Ddocker.compose.skip=true -Ddocker.build.skip=false -pl :hudi-hadoop-hive-docker -am

报错
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
at org.apache.hadoop.conf.Configuration.set(Configuration.java:1357)
解决办法

$ rm /opt/shared/apache-hive-3.1.2-bin/lib/guava-19.0.jar
$ cp /opt/shared/hadoop-3.2.1/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/shared/apache-hive-3.1.2-bin/lib/