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


/opt/hive/lib/hive-metastore-3.1.3.jar::/opt/hive/lib/hive-service-3.1.3.jar::/opt/hive/lib/hive-exec-3.1.3.jar::/opt/hive/lib/hive-jdbc-3.1.3.jar:/opt/hive/lib/hive-jdbc-handler-3.1.3.jar::/opt/hive/lib/jackson-annotations-2.12.0.jar:/opt/hive/lib/jackson-core-2.12.0.jar:/opt/hive/lib/jackson-core-asl-1.9.13.jar:/opt/hive/lib/jackson-databind-2.12.0.jar:/opt/hive/lib/jackson-dataformat-smile-2.12.0.jar:/opt/hive/lib/jackson-mapper-asl-1.9.13.jar:/opt/hive/lib/jackson-module-scala_2.11-2.12.0.jar::/opt/hadoop-3.2.1/share/hadoop/common/*:/opt/hadoop-3.2.1/share/hadoop/mapreduce/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/*:/opt/hadoop-3.2.1/share/hadoop/common/lib/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/lib/*:/etc/hadoop:/var/hoodie/ws/hudi-sync/hudi-hive-sync/../../packaging/hudi-hive-sync-bundle/target/hudi-hive-sync-bundle-1.0.0-SNAPSHOT.jar




java -cp /opt/hive/lib/hive-metastore-3.1.3.jar::/opt/hive/lib/hive-service-3.1.3.jar::/opt/hive/lib/hive-exec-3.1.3.jar::/opt/hive/lib/hive-jdbc-3.1.3.jar:/opt/hive/lib/hive-jdbc-handler-3.1.3.jar::/opt/hive/lib/jackson-annotations-2.12.0.jar:/opt/hive/lib/jackson-core-2.12.0.jar:/opt/hive/lib/jackson-core-asl-1.9.13.jar:/opt/hive/lib/jackson-databind-2.12.0.jar:/opt/hive/lib/jackson-dataformat-smile-2.12.0.jar:/opt/hive/lib/jackson-mapper-asl-1.9.13.jar:/opt/hive/lib/jackson-module-scala_2.11-2.12.0.jar::/opt/hadoop-3.2.1/share/hadoop/common/*:/opt/hadoop-3.2.1/share/hadoop/mapreduce/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/*:/opt/hadoop-3.2.1/share/hadoop/common/lib/*:/opt/hadoop-3.2.1/share/hadoop/hdfs/lib/*:/etc/hadoop:/var/hoodie/ws/hudi-sync/hudi-hive-sync/../../packaging/hudi-hive-sync-bundle/target/hudi-hive-sync-bundle-1.0.0-SNAPSHOT.jar org.apache.hudi.hive.HiveSyncTool --jdbc-url jdbc:hive2://hiveserver:10000 --user hive --pass hive --partitioned-by dt --base-path /user/hive/warehouse/stock_ticks_cow --database default --table stock_ticks_cow --partition-value-extractor org.apache.hudi.hive.SlashEncodedDayPartitionValueExtractor



## A Demo using Docker containers[](https://hudi.apache.org/docs/docker_demo/#a-demo-using-docker-containers "Direct link to heading")

Let's use a real world example to see how Hudi works end to end. For this purpose, a self contained data infrastructure is brought up in a local Docker cluster within your computer. It requires the Hudi repo to have been cloned locally.

The steps have been tested on a Mac laptop

### Prerequisites[](https://hudi.apache.org/docs/docker_demo/#prerequisites "Direct link to heading")

-   Clone the [Hudi repository](https://github.com/apache/hudi) to your local machine.
    
-   Docker Setup : For Mac, Please follow the steps as defined in [Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/). For running Spark-SQL queries, please ensure atleast 6 GB and 4 CPUs are allocated to Docker (See Docker -> Preferences -> Advanced). Otherwise, spark-SQL queries could be killed because of memory issues.
    
-   kcat : A command-line utility to publish/consume from kafka topics. Use `brew install kcat` to install kcat.
    
-   /etc/hosts : The demo references many services running in container by the hostname. Add the following settings to /etc/hosts
    
    ```
    127.0.0.1 adhoc-1127.0.0.1 adhoc-2127.0.0.1 namenode127.0.0.1 datanode1127.0.0.1 hiveserver127.0.0.1 hivemetastore127.0.0.1 kafkabroker127.0.0.1 sparkmaster127.0.0.1 zookeeper
    ```
    
-   Java : Java SE Development Kit 8.
    
-   Maven : A build automation tool for Java projects.
    
-   jq : A lightweight and flexible command-line JSON processor. Use `brew install jq` to install jq.
    

Also, this has not been tested on some environments like Docker on Windows.

## Setting up Docker Cluster[](https://hudi.apache.org/docs/docker_demo/#setting-up-docker-cluster "Direct link to heading")

### Build Hudi[](https://hudi.apache.org/docs/docker_demo/#build-hudi "Direct link to heading")

The first step is to build Hudi. **Note** This step builds Hudi on default supported scala version - 2.11.

NOTE: Make sure you've cloned the [Hudi repository](https://github.com/apache/hudi) first.

```
cd <HUDI_WORKSPACE>mvn clean package -Pintegration-tests -DskipTests
```

### Bringing up Demo Cluster[](https://hudi.apache.org/docs/docker_demo/#bringing-up-demo-cluster "Direct link to heading")

The next step is to run the Docker compose script and setup configs for bringing up the cluster. These files are in the [Hudi repository](https://github.com/apache/hudi) which you should already have locally on your machine from the previous steps.

This should pull the Docker images from Docker hub and setup the Docker cluster.

-   Default
-   Mac AArch64

```
cd docker./setup_demo.sh............[+] Running 10/13⠿ Container zookeeper             Removed                 8.6s⠿ Container datanode1             Removed                18.3s⠿ Container trino-worker-1        Removed                50.7s⠿ Container spark-worker-1        Removed                16.7s⠿ Container adhoc-2               Removed                16.9s⠿ Container graphite              Removed                16.9s⠿ Container kafkabroker           Removed                14.1s⠿ Container adhoc-1               Removed                14.1s⠿ Container presto-worker-1       Removed                11.9s⠿ Container presto-coordinator-1  Removed                34.6s.............[+] Running 17/17⠿ adhoc-1 Pulled                                          2.9s⠿ graphite Pulled                                         2.8s⠿ spark-worker-1 Pulled                                   3.0s⠿ kafka Pulled                                            2.9s⠿ datanode1 Pulled                                        2.9s⠿ hivemetastore Pulled                                    2.9s⠿ hiveserver Pulled                                       3.0s⠿ hive-metastore-postgresql Pulled                        2.8s⠿ presto-coordinator-1 Pulled                             2.9s⠿ namenode Pulled                                         2.9s⠿ trino-worker-1 Pulled                                   2.9s⠿ sparkmaster Pulled                                      2.9s⠿ presto-worker-1 Pulled                                  2.9s⠿ zookeeper Pulled                                        2.8s⠿ adhoc-2 Pulled                                          2.9s⠿ historyserver Pulled                                    2.9s⠿ trino-coordinator-1 Pulled                              2.9s[+] Running 17/17⠿ Container zookeeper                  Started           41.0s⠿ Container kafkabroker                Started           41.7s⠿ Container graphite                   Started           41.5s⠿ Container hive-metastore-postgresql  Running            0.0s⠿ Container namenode                   Running            0.0s⠿ Container hivemetastore              Running            0.0s⠿ Container trino-coordinator-1        Runni...           0.0s⠿ Container presto-coordinator-1       Star...           42.1s⠿ Container historyserver              Started           41.0s⠿ Container datanode1                  Started           49.9s⠿ Container hiveserver                 Running            0.0s⠿ Container trino-worker-1             Started           42.1s⠿ Container sparkmaster                Started           41.9s⠿ Container spark-worker-1             Started           50.2s⠿ Container adhoc-2                    Started           38.5s⠿ Container adhoc-1                    Started           38.5s⠿ Container presto-worker-1            Started           38.4sCopying spark default config and setting up configsCopying spark default config and setting up configs$ docker ps
```

At this point, the Docker cluster will be up and running. The demo cluster brings up the following services

-   HDFS Services (NameNode, DataNode)
-   Spark Master and Worker
-   Hive Services (Metastore, HiveServer2 along with PostgresDB)
-   Kafka Broker and a Zookeeper Node (Kafka will be used as upstream source for the demo)
-   Containers for Presto setup (Presto coordinator and worker)
-   Containers for Trino setup (Trino coordinator and worker)
-   Adhoc containers to run Hudi/Hive CLI commands

## Demo[](https://hudi.apache.org/docs/docker_demo/#demo "Direct link to heading")

Stock Tracker data will be used to showcase different Hudi query types and the effects of Compaction.

Take a look at the directory `docker/demo/data`. There are 2 batches of stock data - each at 1 minute granularity. The first batch contains stocker tracker data for some stock symbols during the first hour of trading window (9:30 a.m to 10:30 a.m). The second batch contains tracker data for next 30 mins (10:30 - 11 a.m). Hudi will be used to ingest these batches to a table which will contain the latest stock tracker data at hour level granularity. The batches are windowed intentionally so that the second batch contains updates to some of the rows in the first batch.

### Step 1 : Publish the first batch to Kafka[](https://hudi.apache.org/docs/docker_demo/#step-1--publish-the-first-batch-to-kafka "Direct link to heading")

Upload the first batch to Kafka topic 'stock ticks'

`cat docker/demo/data/batch_1.json | kcat -b kafkabroker -t stock_ticks -P`

To check if the new topic shows up, use

```
kcat -b kafkabroker -L -J | jq .{  "originating_broker": {    "id": 1001,    "name": "kafkabroker:9092/1001"  },  "query": {    "topic": "*"  },  "brokers": [    {      "id": 1001,      "name": "kafkabroker:9092"    }  ],  "topics": [    {      "topic": "stock_ticks",      "partitions": [        {          "partition": 0,          "leader": 1001,          "replicas": [            {              "id": 1001            }          ],          "isrs": [            {              "id": 1001            }          ]        }      ]    }  ]}
```

### Step 2: Incrementally ingest data from Kafka topic[](https://hudi.apache.org/docs/docker_demo/#step-2-incrementally-ingest-data-from-kafka-topic "Direct link to heading")

Hudi comes with a tool named Hudi Streamer. This tool can connect to variety of data sources (including Kafka) to pull changes and apply to Hudi table using upsert/insert primitives. Here, we will use the tool to download json data from kafka topic and ingest to both COW and MOR tables we initialized in the previous step. This tool automatically initializes the tables in the file-system if they do not exist yet.

```
docker exec -it adhoc-2 /bin/bash# Run the following spark-submit command to execute the Hudi Streamer and ingest to stock_ticks_cow table in HDFSspark-submit \  --class org.apache.hudi.utilities.streamer.HoodieStreamer $HUDI_UTILITIES_BUNDLE \  --table-type COPY_ON_WRITE \  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \  --source-ordering-field ts  \  --target-base-path /user/hive/warehouse/stock_ticks_cow \  --target-table stock_ticks_cow --props /var/demo/config/kafka-source.properties \  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider# Run the following spark-submit command to execute the Hudi Streamer and ingest to stock_ticks_mor table in HDFSspark-submit \  --class org.apache.hudi.utilities.streamer.HoodieStreamer $HUDI_UTILITIES_BUNDLE \  --table-type MERGE_ON_READ \  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \  --source-ordering-field ts \  --target-base-path /user/hive/warehouse/stock_ticks_mor \  --target-table stock_ticks_mor \  --props /var/demo/config/kafka-source.properties \  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \  --disable-compaction# As part of the setup (Look at setup_demo.sh), the configs needed for Hudi Streamer is uploaded to HDFS. The configs# contain mostly Kafa connectivity settings, the avro-schema to be used for ingesting along with key and partitioning fields.exit
```

You can use HDFS web-browser to look at the tables `http://namenode:50070/explorer.html#/user/hive/warehouse/stock_ticks_cow`.

You can explore the new partition folder created in the table along with a "commit" / "deltacommit" file under .hoodie which signals a successful commit.

There will be a similar setup when you browse the MOR table `http://namenode:50070/explorer.html#/user/hive/warehouse/stock_ticks_mor`

### Step 3: Sync with Hive[](https://hudi.apache.org/docs/docker_demo/#step-3-sync-with-hive "Direct link to heading")

At this step, the tables are available in HDFS. We need to sync with Hive to create new Hive tables and add partitions inorder to run Hive queries against those tables.

```
docker exec -it adhoc-2 /bin/bash# This command takes in HiveServer URL and COW Hudi table location in HDFS and sync the HDFS state to Hive/var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \  --jdbc-url jdbc:hive2://hiveserver:10000 \  --user hive \  --pass hive \  --partitioned-by dt \  --base-path /user/hive/warehouse/stock_ticks_cow \  --database default \  --table stock_ticks_cow \  --partition-value-extractor org.apache.hudi.hive.SlashEncodedDayPartitionValueExtractor.....2020-01-25 19:51:28,953 INFO  [main] hive.HiveSyncTool (HiveSyncTool.java:syncHoodieTable(129)) - Sync complete for stock_ticks_cow.....# Now run hive-sync for the second data-set in HDFS using Merge-On-Read (MOR table type)/var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \  --jdbc-url jdbc:hive2://hiveserver:10000 \  --user hive \  --pass hive \  --partitioned-by dt \  --base-path /user/hive/warehouse/stock_ticks_mor \  --database default \  --table stock_ticks_mor \  --partition-value-extractor org.apache.hudi.hive.SlashEncodedDayPartitionValueExtractor...2020-01-25 19:51:51,066 INFO  [main] hive.HiveSyncTool (HiveSyncTool.java:syncHoodieTable(129)) - Sync complete for stock_ticks_mor_ro...2020-01-25 19:51:51,569 INFO  [main] hive.HiveSyncTool (HiveSyncTool.java:syncHoodieTable(129)) - Sync complete for stock_ticks_mor_rt....exit
```

After executing the above command, you will notice

1.  A hive table named `stock_ticks_cow` created which supports Snapshot and Incremental queries on Copy On Write table.
2.  Two new tables `stock_ticks_mor_rt` and `stock_ticks_mor_ro` created for the Merge On Read table. The former supports Snapshot and Incremental queries (providing near-real time data) while the later supports ReadOptimized queries.

### Step 4 (a): Run Hive Queries[](https://hudi.apache.org/docs/docker_demo/#step-4-a-run-hive-queries "Direct link to heading")

Run a hive query to find the latest timestamp ingested for stock symbol 'GOOG'. You will notice that both snapshot (for both COW and MOR \_rt table) and read-optimized queries (for MOR \_ro table) give the same value "10:29 a.m" as Hudi create a parquet file for the first batch of data.

```
docker exec -it adhoc-2 /bin/bashbeeline -u jdbc:hive2://hiveserver:10000 \  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \  --hiveconf hive.stats.autogather=false# List Tables0: jdbc:hive2://hiveserver:10000> show tables;+---------------------+--+|      tab_name       |+---------------------+--+| stock_ticks_cow     || stock_ticks_mor_ro  || stock_ticks_mor_rt  |+---------------------+--+3 rows selected (1.199 seconds)0: jdbc:hive2://hiveserver:10000># Look at partitions that were added0: jdbc:hive2://hiveserver:10000> show partitions stock_ticks_mor_rt;+----------------+--+|   partition    |+----------------+--+| dt=2018-08-31  |+----------------+--+1 row selected (0.24 seconds)# COPY-ON-WRITE Queries:=========================0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:29:00  |+---------+----------------------+--+Now, run a projection query:0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924221953       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924221953       | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |+----------------------+---------+----------------------+---------+------------+-----------+--+# Merge-On-Read Queries:==========================Lets run similar queries against M-O-R table. Lets look at both ReadOptimized and Snapshot(realtime data) queries supported by M-O-R table# Run ReadOptimized Query. Notice that the latest timestamp is 10:290: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:29:00  |+---------+----------------------+--+1 row selected (6.326 seconds)# Run Snapshot Query. Notice that the latest timestamp is again 10:290: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:29:00  |+---------+----------------------+--+1 row selected (1.606 seconds)# Run Read Optimized and Snapshot project queries0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924222155       | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |+----------------------+---------+----------------------+---------+------------+-----------+--+0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924222155       | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |+----------------------+---------+----------------------+---------+------------+-----------+--+exit
```

### Step 4 (b): Run Spark-SQL Queries[](https://hudi.apache.org/docs/docker_demo/#step-4-b-run-spark-sql-queries "Direct link to heading")

Hudi support Spark as query processor just like Hive. Here are the same hive queries running in spark-sql

```
docker exec -it adhoc-1 /bin/bash$SPARK_INSTALL/bin/spark-shell \  --jars $HUDI_SPARK_BUNDLE \  --master local[2] \  --driver-class-path $HADOOP_CONF_DIR \  --conf spark.sql.hive.convertMetastoreParquet=false \  --deploy-mode client \  --driver-memory 1G \  --executor-memory 3G \  --num-executors 1...Welcome to      ____              __     / __/__  ___ _____/ /__    _\ \/ _ \/ _ `/ __/  '_/   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4      /_/Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)Type in expressions to have them evaluated.Type :help for more information.scala> spark.sql("show tables").show(100, false)+--------+------------------+-----------+|database|tableName         |isTemporary|+--------+------------------+-----------+|default |stock_ticks_cow   |false      ||default |stock_ticks_mor_ro|false      ||default |stock_ticks_mor_rt|false      |+--------+------------------+-----------+# Copy-On-Write Table## Run max timestamp query against COW tablescala> spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)[Stage 0:>                                                          (0 + 1) / 1]SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".SLF4J: Defaulting to no-operation (NOP) logger implementationSLF4J: See http://www.slf4j.org/codes#StaticLoggerBinder for further details.+------+-------------------+|symbol|max(ts)            |+------+-------------------+|GOOG  |2018-08-31 10:29:00|+------+-------------------+## Projection Queryscala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)+-------------------+------+-------------------+------+---------+--------+|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |+-------------------+------+-------------------+------+---------+--------+|20180924221953     |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 ||20180924221953     |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|+-------------------+------+-------------------+------+---------+--------+# Merge-On-Read Queries:==========================Lets run similar queries against M-O-R table. Lets look at bothReadOptimized and Snapshot queries supported by M-O-R table# Run ReadOptimized Query. Notice that the latest timestamp is 10:29scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)+------+-------------------+|symbol|max(ts)            |+------+-------------------+|GOOG  |2018-08-31 10:29:00|+------+-------------------+# Run Snapshot Query. Notice that the latest timestamp is again 10:29scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)+------+-------------------+|symbol|max(ts)            |+------+-------------------+|GOOG  |2018-08-31 10:29:00|+------+-------------------+# Run Read Optimized and Snapshot project queriesscala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)+-------------------+------+-------------------+------+---------+--------+|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |+-------------------+------+-------------------+------+---------+--------+|20180924222155     |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 ||20180924222155     |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|+-------------------+------+-------------------+------+---------+--------+scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)+-------------------+------+-------------------+------+---------+--------+|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |+-------------------+------+-------------------+------+---------+--------+|20180924222155     |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 ||20180924222155     |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|+-------------------+------+-------------------+------+---------+--------+
```

### Step 4 (c): Run Presto Queries[](https://hudi.apache.org/docs/docker_demo/#step-4-c-run-presto-queries "Direct link to heading")

Here are the Presto queries for similar Hive and Spark queries.

##### note

-   Currently, Presto does not support snapshot or incremental queries on Hudi tables.
-   This section of the demo is not supported for Mac AArch64 users at this time.

```
docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090presto> show catalogs;  Catalog----------- hive jmx localfile system(4 rows)Query 20190817_134851_00000_j8rcz, FINISHED, 1 nodeSplits: 19 total, 19 done (100.00%)0:04 [0 rows, 0B] [0 rows/s, 0B/s]presto> use hive.default;USEpresto:default> show tables;       Table-------------------- stock_ticks_cow stock_ticks_mor_ro stock_ticks_mor_rt(3 rows)Query 20190822_181000_00001_segyw, FINISHED, 2 nodesSplits: 19 total, 19 done (100.00%)0:05 [3 rows, 99B] [0 rows/s, 18B/s]# COPY-ON-WRITE Queries:=========================presto:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'; symbol |        _col1--------+--------------------- GOOG   | 2018-08-31 10:29:00(1 row)Query 20190822_181011_00002_segyw, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0:12 [197 rows, 613B] [16 rows/s, 50B/s]presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close---------------------+--------+---------------------+--------+-----------+---------- 20190822180221      | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 20190822180221      | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085(2 rows)Query 20190822_181141_00003_segyw, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0:02 [197 rows, 613B] [109 rows/s, 341B/s]# Merge-On-Read Queries:==========================Lets run similar queries against M-O-R table. # Run ReadOptimized Query. Notice that the latest timestamp is 10:29    presto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'; symbol |        _col1--------+--------------------- GOOG   | 2018-08-31 10:29:00(1 row)Query 20190822_181158_00004_segyw, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0:02 [197 rows, 613B] [110 rows/s, 343B/s]presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close---------------------+--------+---------------------+--------+-----------+---------- 20190822180250      | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 20190822180250      | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085(2 rows)Query 20190822_181256_00006_segyw, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0:02 [197 rows, 613B] [92 rows/s, 286B/s]presto:default> exit
```

### Step 4 (d): Run Trino Queries[](https://hudi.apache.org/docs/docker_demo/#step-4-d-run-trino-queries "Direct link to heading")

Here are the similar queries with Trino.

##### note

-   Currently, Trino does not support snapshot or incremental queries on Hudi tables.
-   This section of the demo is not supported for Mac AArch64 users at this time.

```
docker exec -it adhoc-2 trino --server trino-coordinator-1:8091trino> show catalogs; Catalog --------- hive     system  (2 rows)Query 20220112_055038_00000_sac73, FINISHED, 1 nodeSplits: 19 total, 19 done (100.00%)3.74 [0 rows, 0B] [0 rows/s, 0B/s]trino> use hive.default;USEtrino:default> show tables;       Table        -------------------- stock_ticks_cow     stock_ticks_mor_ro  stock_ticks_mor_rt (3 rows)Query 20220112_055050_00003_sac73, FINISHED, 2 nodesSplits: 19 total, 19 done (100.00%)1.84 [3 rows, 102B] [1 rows/s, 55B/s]# COPY-ON-WRITE Queries:=========================    trino:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'; symbol |        _col1        --------+--------------------- GOOG   | 2018-08-31 10:29:00 (1 row)Query 20220112_055101_00005_sac73, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)4.08 [197 rows, 442KB] [48 rows/s, 108KB/s]trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   ---------------------+--------+---------------------+--------+-----------+---------- 20220112054822108   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02  20220112054822108   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 (2 rows)Query 20220112_055113_00006_sac73, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0.40 [197 rows, 450KB] [487 rows/s, 1.09MB/s]# Merge-On-Read Queries:==========================Lets run similar queries against MOR table.# Run ReadOptimized Query. Notice that the latest timestamp is 10:29    trino:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'; symbol |        _col1        --------+--------------------- GOOG   | 2018-08-31 10:29:00 (1 row)Query 20220112_055125_00007_sac73, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0.50 [197 rows, 442KB] [395 rows/s, 888KB/s]trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   ---------------------+--------+---------------------+--------+-----------+---------- 20220112054844841   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02  20220112054844841   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 (2 rows)Query 20220112_055136_00008_sac73, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0.49 [197 rows, 450KB] [404 rows/s, 924KB/s]trino:default> exit
```

### Step 5: Upload second batch to Kafka and run Hudi Streamer to ingest[](https://hudi.apache.org/docs/docker_demo/#step-5-upload-second-batch-to-kafka-and-run-hudi-streamer-to-ingest "Direct link to heading")

Upload the second batch of data and ingest this batch using Hudi Streamer. As this batch does not bring in any new partitions, there is no need to run hive-sync

```
cat docker/demo/data/batch_2.json | kcat -b kafkabroker -t stock_ticks -P# Within Docker container, run the ingestion commanddocker exec -it adhoc-2 /bin/bash# Run the following spark-submit command to execute the Hudi Streamer and ingest to stock_ticks_cow table in HDFSspark-submit \  --class org.apache.hudi.utilities.streamer.HoodieStreamer $HUDI_UTILITIES_BUNDLE \  --table-type COPY_ON_WRITE \  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \  --source-ordering-field ts \  --target-base-path /user/hive/warehouse/stock_ticks_cow \  --target-table stock_ticks_cow \  --props /var/demo/config/kafka-source.properties \  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider# Run the following spark-submit command to execute the Hudi Streamer and ingest to stock_ticks_mor table in HDFSspark-submit \  --class org.apache.hudi.utilities.streamer.HoodieStreamer $HUDI_UTILITIES_BUNDLE \  --table-type MERGE_ON_READ \  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \  --source-ordering-field ts \  --target-base-path /user/hive/warehouse/stock_ticks_mor \  --target-table stock_ticks_mor \  --props /var/demo/config/kafka-source.properties \  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \  --disable-compactionexit
```

With Copy-On-Write table, the second ingestion by Hudi Streamer resulted in a new version of Parquet file getting created. See `http://namenode:50070/explorer.html#/user/hive/warehouse/stock_ticks_cow/2018/08/31`

With Merge-On-Read table, the second ingestion merely appended the batch to an unmerged delta (log) file. Take a look at the HDFS filesystem to get an idea: `http://namenode:50070/explorer.html#/user/hive/warehouse/stock_ticks_mor/2018/08/31`

### Step 6 (a): Run Hive Queries[](https://hudi.apache.org/docs/docker_demo/#step-6-a-run-hive-queries "Direct link to heading")

With Copy-On-Write table, the Snapshot query immediately sees the changes as part of second batch once the batch got committed as each ingestion creates newer versions of parquet files.

With Merge-On-Read table, the second ingestion merely appended the batch to an unmerged delta (log) file. This is the time, when ReadOptimized and Snapshot queries will provide different results. ReadOptimized query will still return "10:29 am" as it will only read from the Parquet file. Snapshot query will do on-the-fly merge and return latest committed data which is "10:59 a.m".

```
docker exec -it adhoc-2 /bin/bashbeeline -u jdbc:hive2://hiveserver:10000 \  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \  --hiveconf hive.stats.autogather=false# Copy On Write Table:0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+--+1 row selected (1.932 seconds)0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924221953       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924224524       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.# Merge On Read Table:# Read Optimized Query0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:29:00  |+---------+----------------------+--+1 row selected (1.6 seconds)0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924222155       | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |+----------------------+---------+----------------------+---------+------------+-----------+--+# Snapshot Query0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+--+0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924224537       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+exit
```

### Step 6 (b): Run Spark SQL Queries[](https://hudi.apache.org/docs/docker_demo/#step-6-b-run-spark-sql-queries "Direct link to heading")

Running the same queries in Spark-SQL:

```
docker exec -it adhoc-1 /bin/bash$SPARK_INSTALL/bin/spark-shell \  --jars $HUDI_SPARK_BUNDLE \  --driver-class-path $HADOOP_CONF_DIR \  --conf spark.sql.hive.convertMetastoreParquet=false \  --deploy-mode client \  --driver-memory 1G \  --master local[2] \  --executor-memory 3G \  --num-executors 1# Copy On Write Table:scala> spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)+------+-------------------+|symbol|max(ts)            |+------+-------------------+|GOOG  |2018-08-31 10:59:00|+------+-------------------+scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924221953       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924224524       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.# Merge On Read Table:# Read Optimized Queryscala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)+---------+----------------------+| symbol  |         _c1          |+---------+----------------------+| GOOG    | 2018-08-31 10:29:00  |+---------+----------------------+1 row selected (1.6 seconds)scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)+----------------------+---------+----------------------+---------+------------+-----------+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924222155       | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |+----------------------+---------+----------------------+---------+------------+-----------+# Snapshot Queryscala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)+---------+----------------------+| symbol  |         _c1          |+---------+----------------------+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)+----------------------+---------+----------------------+---------+------------+-----------+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+| 20180924222155       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924224537       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+exit
```

### Step 6 (c): Run Presto Queries[](https://hudi.apache.org/docs/docker_demo/#step-6-c-run-presto-queries "Direct link to heading")

Running the same queries on Presto for ReadOptimized queries.

##### note

This section of the demo is not supported for Mac AArch64 users at this time.

```
docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090presto> use hive.default;USE# Copy On Write Table:presto:default>select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'; symbol |        _col1--------+--------------------- GOOG   | 2018-08-31 10:59:00(1 row)Query 20190822_181530_00007_segyw, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0:02 [197 rows, 613B] [125 rows/s, 389B/s]presto:default>select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close---------------------+--------+---------------------+--------+-----------+---------- 20190822180221      | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 20190822181433      | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215(2 rows)Query 20190822_181545_00008_segyw, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0:02 [197 rows, 613B] [106 rows/s, 332B/s]As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.# Merge On Read Table:# Read Optimized Querypresto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'; symbol |        _col1--------+--------------------- GOOG   | 2018-08-31 10:29:00(1 row)Query 20190822_181602_00009_segyw, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0:01 [197 rows, 613B] [139 rows/s, 435B/s]presto:default>select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close---------------------+--------+---------------------+--------+-----------+---------- 20190822180250      | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 20190822180250      | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085(2 rows)Query 20190822_181615_00010_segyw, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0:01 [197 rows, 613B] [154 rows/s, 480B/s]presto:default> exit
```

### Step 6 (d): Run Trino Queries[](https://hudi.apache.org/docs/docker_demo/#step-6-d-run-trino-queries "Direct link to heading")

Running the same queries on Trino for Read-Optimized queries.

##### note

This section of the demo is not supported for Mac AArch64 users at this time.

```
docker exec -it adhoc-2 trino --server trino-coordinator-1:8091trino> use hive.default;USE    # Copy On Write Table:trino:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'; symbol |        _col1        --------+--------------------- GOOG   | 2018-08-31 10:59:00 (1 row)Query 20220112_055443_00012_sac73, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0.63 [197 rows, 442KB] [310 rows/s, 697KB/s]trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   ---------------------+--------+---------------------+--------+-----------+---------- 20220112054822108   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02  20220112055352654   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 (2 rows)Query 20220112_055450_00013_sac73, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0.65 [197 rows, 450KB] [303 rows/s, 692KB/s]As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.# Merge On Read Table:# Read Optimized Query    trino:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'; symbol |        _col1        --------+--------------------- GOOG   | 2018-08-31 10:29:00 (1 row)Query 20220112_055500_00014_sac73, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0.59 [197 rows, 442KB] [336 rows/s, 756KB/s]trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   ---------------------+--------+---------------------+--------+-----------+---------- 20220112054844841   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02  20220112054844841   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 (2 rows)Query 20220112_055506_00015_sac73, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0.35 [197 rows, 450KB] [556 rows/s, 1.24MB/s]trino:default> exit
```

### Step 7 (a): Incremental Query for COPY-ON-WRITE Table[](https://hudi.apache.org/docs/docker_demo/#step-7-a-incremental-query-for-copy-on-write-table "Direct link to heading")

With 2 batches of data ingested, lets showcase the support for incremental queries in Hudi Copy-On-Write tables

Lets take the same projection query example

```
docker exec -it adhoc-2 /bin/bashbeeline -u jdbc:hive2://hiveserver:10000 \  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \  --hiveconf hive.stats.autogather=false0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924064621       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924065039       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+
```

As you notice from the above queries, there are 2 commits - 20180924064621 and 20180924065039 in timeline order. When you follow the steps, you will be getting different timestamps for commits. Substitute them in place of the above timestamps.

To show the effects of incremental-query, let us assume that a reader has already seen the changes as part of ingesting first batch. Now, for the reader to see effect of the second batch, he/she has to keep the start timestamp to the commit time of the first batch (20180924064621) and run incremental query

Hudi incremental mode provides efficient scanning for incremental queries by filtering out files that do not have any candidate rows using hudi-managed metadata.

```
docker exec -it adhoc-2 /bin/bashbeeline -u jdbc:hive2://hiveserver:10000 \  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \  --hiveconf hive.stats.autogather=false0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_cow.consume.mode=INCREMENTAL;No rows affected (0.009 seconds)0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_cow.consume.max.commits=3;No rows affected (0.009 seconds)0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_cow.consume.start.timestamp=20180924064621;
```

With the above setting, file-ids that do not have any updates from the commit 20180924065039 is filtered out without scanning. Here is the incremental query :

```
0: jdbc:hive2://hiveserver:10000>0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG' and `_hoodie_commit_time` > '20180924064621';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924065039       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+1 row selected (0.83 seconds)0: jdbc:hive2://hiveserver:10000>
```

### Step 7 (b): Incremental Query with Spark SQL:[](https://hudi.apache.org/docs/docker_demo/#step-7-b-incremental-query-with-spark-sql "Direct link to heading")

```
docker exec -it adhoc-1 /bin/bash$SPARK_INSTALL/bin/spark-shell \  --jars $HUDI_SPARK_BUNDLE \  --driver-class-path $HADOOP_CONF_DIR \  --conf spark.sql.hive.convertMetastoreParquet=false \  --deploy-mode client \  --driver-memory 1G \  --master local[2] \  --executor-memory 3G \  --num-executors 1Welcome to      ____              __     / __/__  ___ _____/ /__    _\ \/ _ \/ _ `/ __/  '_/   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4      /_/Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)Type in expressions to have them evaluated.Type :help for more information.scala> import org.apache.hudi.DataSourceReadOptionsimport org.apache.hudi.DataSourceReadOptions# In the below query, 20180925045257 is the first commit's timestampscala> val hoodieIncViewDF =  spark.read.format("org.apache.hudi").option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY, DataSourceReadOptions.QUERY_TYPE_INCREMENTAL_OPT_VAL).option(DataSourceReadOptions.BEGIN_INSTANTTIME_OPT_KEY, "20180924064621").load("/user/hive/warehouse/stock_ticks_cow")SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".SLF4J: Defaulting to no-operation (NOP) logger implementationSLF4J: See http://www.slf4j.org/codes#StaticLoggerBinder for further details.hoodieIncViewDF: org.apache.spark.sql.DataFrame = [_hoodie_commit_time: string, _hoodie_commit_seqno: string ... 15 more fields]scala> hoodieIncViewDF.registerTempTable("stock_ticks_cow_incr_tmp1")warning: there was one deprecation warning; re-run with -deprecation for detailsscala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow_incr_tmp1 where  symbol = 'GOOG'").show(100, false);+----------------------+---------+----------------------+---------+------------+-----------+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+| 20180924065039       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+
```

### Step 8: Schedule and Run Compaction for Merge-On-Read table[](https://hudi.apache.org/docs/docker_demo/#step-8-schedule-and-run-compaction-for-merge-on-read-table "Direct link to heading")

Lets schedule and run a compaction to create a new version of columnar file so that read-optimized readers will see fresher data. Again, You can use Hudi CLI to manually schedule and run compaction

```
docker exec -it adhoc-1 /bin/bashroot@adhoc-1:/opt# /var/hoodie/ws/hudi-cli/hudi-cli.sh...Table command getting loadedHoodieSplashScreen loaded===================================================================*         ___                          ___                        **        /\__\          ___           /\  \           ___         **       / /  /         /\__\         /  \  \         /\  \        **      / /__/         / /  /        / /\ \  \        \ \  \       **     /  \  \ ___    / /  /        / /  \ \__\       /  \__\      **    / /\ \  /\__\  / /__/  ___   / /__/ \ |__|     / /\/__/      **    \/  \ \/ /  /  \ \  \ /\__\  \ \  \ / /  /  /\/ /  /         **         \  /  /    \ \  / /  /   \ \  / /  /   \  /__/          **         / /  /      \ \/ /  /     \ \/ /  /     \ \__\          **        / /  /        \  /  /       \  /  /       \/__/          **        \/__/          \/__/         \/__/    Apache Hudi CLI    **                                                                 *===================================================================Welcome to Apache Hudi CLI. Please type help if you are looking for help.hudi->connect --path /user/hive/warehouse/stock_ticks_mor18/09/24 06:59:34 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable18/09/24 06:59:35 INFO table.HoodieTableMetaClient: Loading HoodieTableMetaClient from /user/hive/warehouse/stock_ticks_mor18/09/24 06:59:35 INFO util.FSUtils: Hadoop Configuration: fs.defaultFS: [hdfs://namenode:8020], Config:[Configuration: core-default.xml, core-site.xml, mapred-default.xml, mapred-site.xml, yarn-default.xml, yarn-site.xml, hdfs-default.xml, hdfs-site.xml], FileSystem: [DFS[DFSClient[clientName=DFSClient_NONMAPREDUCE_-1261652683_11, ugi=root (auth:SIMPLE)]]]18/09/24 06:59:35 INFO table.HoodieTableConfig: Loading table properties from /user/hive/warehouse/stock_ticks_mor/.hoodie/hoodie.properties18/09/24 06:59:36 INFO table.HoodieTableMetaClient: Finished Loading Table of type MERGE_ON_READ(version=1) from /user/hive/warehouse/stock_ticks_morMetadata for table stock_ticks_mor loadedhoodie:stock_ticks_mor->compactions show all20/02/10 03:41:32 INFO timeline.HoodieActiveTimeline: Loaded instants [[20200210015059__clean__COMPLETED], [20200210015059__deltacommit__COMPLETED], [20200210022758__clean__COMPLETED], [20200210022758__deltacommit__COMPLETED], [==>20200210023843__compaction__REQUESTED]]___________________________________________________________________| Compaction Instant Time| State    | Total FileIds to be Compacted||==================================================================|# Schedule a compaction. This will use Spark Launcher to schedule compactionhoodie:stock_ticks_mor->compaction schedule --hoodieConfigs hoodie.compact.inline.max.delta.commits=1....Compaction successfully completed for 20180924070031# Now refresh and check again. You will see that there is a new compaction requestedhoodie:stock_ticks_mor->refresh18/09/24 07:01:16 INFO table.HoodieTableMetaClient: Loading HoodieTableMetaClient from /user/hive/warehouse/stock_ticks_mor18/09/24 07:01:16 INFO table.HoodieTableConfig: Loading table properties from /user/hive/warehouse/stock_ticks_mor/.hoodie/hoodie.properties18/09/24 07:01:16 INFO table.HoodieTableMetaClient: Finished Loading Table of type MERGE_ON_READ(version=1) from /user/hive/warehouse/stock_ticks_morMetadata for table stock_ticks_mor loadedhoodie:stock_ticks_mor->compactions show all18/09/24 06:34:12 INFO timeline.HoodieActiveTimeline: Loaded instants [[20180924041125__clean__COMPLETED], [20180924041125__deltacommit__COMPLETED], [20180924042735__clean__COMPLETED], [20180924042735__deltacommit__COMPLETED], [==>20180924063245__compaction__REQUESTED]]___________________________________________________________________| Compaction Instant Time| State    | Total FileIds to be Compacted||==================================================================|| 20180924070031         | REQUESTED| 1                            |# Execute the compaction. The compaction instant value passed below must be the one displayed in the above "compactions show all" queryhoodie:stock_ticks_mor->compaction run --compactionInstant  20180924070031 --parallelism 2 --sparkMemory 1G  --schemaFilePath /var/demo/config/schema.avsc --retry 1  ....Compaction successfully completed for 20180924070031## Now check if compaction is completedhoodie:stock_ticks_mor->refresh18/09/24 07:03:00 INFO table.HoodieTableMetaClient: Loading HoodieTableMetaClient from /user/hive/warehouse/stock_ticks_mor18/09/24 07:03:00 INFO table.HoodieTableConfig: Loading table properties from /user/hive/warehouse/stock_ticks_mor/.hoodie/hoodie.properties18/09/24 07:03:00 INFO table.HoodieTableMetaClient: Finished Loading Table of type MERGE_ON_READ(version=1) from /user/hive/warehouse/stock_ticks_morMetadata for table stock_ticks_mor loadedhoodie:stock_ticks_mor->compactions show all18/09/24 07:03:15 INFO timeline.HoodieActiveTimeline: Loaded instants [[20180924064636__clean__COMPLETED], [20180924064636__deltacommit__COMPLETED], [20180924065057__clean__COMPLETED], [20180924065057__deltacommit__COMPLETED], [20180924070031__commit__COMPLETED]]___________________________________________________________________| Compaction Instant Time| State    | Total FileIds to be Compacted||==================================================================|| 20180924070031         | COMPLETED| 1                            |
```

### Step 9: Run Hive Queries including incremental queries[](https://hudi.apache.org/docs/docker_demo/#step-9-run-hive-queries-including-incremental-queries "Direct link to heading")

You will see that both ReadOptimized and Snapshot queries will show the latest committed data. Lets also run the incremental query for MOR table. From looking at the below query output, it will be clear that the fist commit time for the MOR table is 20180924064636 and the second commit time is 20180924070031

```
docker exec -it adhoc-2 /bin/bashbeeline -u jdbc:hive2://hiveserver:10000 \  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \  --hiveconf hive.stats.autogather=false# Read Optimized Query0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+--+1 row selected (1.6 seconds)0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924064636       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924070031       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+# Snapshot Query0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.+---------+----------------------+--+| symbol  |         _c1          |+---------+----------------------+--+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+--+0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924064636       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924070031       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+# Incremental Query:0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_mor.consume.mode=INCREMENTAL;No rows affected (0.008 seconds)# Max-Commits covers both second batch and compaction commit0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_mor.consume.max.commits=3;No rows affected (0.007 seconds)0: jdbc:hive2://hiveserver:10000> set hoodie.stock_ticks_mor.consume.start.timestamp=20180924064636;No rows affected (0.013 seconds)# Query:0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG' and `_hoodie_commit_time` > '20180924064636';+----------------------+---------+----------------------+---------+------------+-----------+--+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+--+| 20180924070031       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+--+exit
```

### Step 10: Read Optimized and Snapshot queries for MOR with Spark-SQL after compaction[](https://hudi.apache.org/docs/docker_demo/#step-10-read-optimized-and-snapshot-queries-for-mor-with-spark-sql-after-compaction "Direct link to heading")

```
docker exec -it adhoc-1 /bin/bash$SPARK_INSTALL/bin/spark-shell \  --jars $HUDI_SPARK_BUNDLE \  --driver-class-path $HADOOP_CONF_DIR \  --conf spark.sql.hive.convertMetastoreParquet=false \  --deploy-mode client \  --driver-memory 1G \  --master local[2] \  --executor-memory 3G \  --num-executors 1# Read Optimized Queryscala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)+---------+----------------------+| symbol  |        max(ts)       |+---------+----------------------+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+1 row selected (1.6 seconds)scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)+----------------------+---------+----------------------+---------+------------+-----------+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+| 20180924064636       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924070031       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+# Snapshot Queryscala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)+---------+----------------------+| symbol  |     max(ts)          |+---------+----------------------+| GOOG    | 2018-08-31 10:59:00  |+---------+----------------------+scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)+----------------------+---------+----------------------+---------+------------+-----------+| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |+----------------------+---------+----------------------+---------+------------+-----------+| 20180924064636       | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   || 20180924070031       | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |+----------------------+---------+----------------------+---------+------------+-----------+
```

### Step 11: Presto Read Optimized queries on MOR table after compaction[](https://hudi.apache.org/docs/docker_demo/#step-11--presto-read-optimized-queries-on-mor-table-after-compaction "Direct link to heading")

##### note

This section of the demo is not supported for Mac AArch64 users at this time.

```
docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090presto> use hive.default;USE# Read Optimized Queryresto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';  symbol |        _col1--------+--------------------- GOOG   | 2018-08-31 10:59:00(1 row)Query 20190822_182319_00011_segyw, FINISHED, 1 nodeSplits: 49 total, 49 done (100.00%)0:01 [197 rows, 613B] [133 rows/s, 414B/s]presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'; _hoodie_commit_time | symbol |         ts          | volume |   open    |  close---------------------+--------+---------------------+--------+-----------+---------- 20190822180250      | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 20190822181944      | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215(2 rows)Query 20190822_182333_00012_segyw, FINISHED, 1 nodeSplits: 17 total, 17 done (100.00%)0:02 [197 rows, 613B] [98 rows/s, 307B/s]presto:default>
```

This brings the demo to an end.

## Testing Hudi in Local Docker environment[](https://hudi.apache.org/docs/docker_demo/#testing-hudi-in-local-docker-environment "Direct link to heading")

You can bring up a Hadoop Docker environment containing Hadoop, Hive and Spark services with support for Hudi.

```
$ mvn pre-integration-test -DskipTests
```

The above command builds Docker images for all the services with current Hudi source installed at /var/hoodie/ws and also brings up the services using a compose file. We currently use Hadoop (v2.8.4), Hive (v2.3.3) and Spark (v2.4.4) in Docker images.

To bring down the containers

```
$ cd hudi-integ-test$ mvn docker-compose:down
```

If you want to bring up the Docker containers, use

```
$ cd hudi-integ-test$ mvn docker-compose:up -DdetachedMode=true
```

Hudi is a library that is operated in a broader data analytics/ingestion environment involving Hadoop, Hive and Spark. Interoperability with all these systems is a key objective for us. We are actively adding integration-tests under **hudi-integ-test/src/test/java** that makes use of this docker environment (See **hudi-integ-test/src/test/java/org/apache/hudi/integ/ITTestHoodieSanity.java** )

### Building Local Docker Containers:[](https://hudi.apache.org/docs/docker_demo/#building-local-docker-containers "Direct link to heading")

The Docker images required for demo and running integration test are already in docker-hub. The Docker images and compose scripts are carefully implemented so that they serve dual-purpose

1.  The Docker images have inbuilt Hudi jar files with environment variable pointing to those jars (HUDI\_HADOOP\_BUNDLE, ...)
2.  For running integration-tests, we need the jars generated locally to be used for running services within docker. The docker-compose scripts (see `docker/compose/docker-compose_hadoop284_hive233_spark244.yml`) ensures local jars override inbuilt jars by mounting local Hudi workspace over the Docker location
3.  As these Docker containers have mounted local Hudi workspace, any changes that happen in the workspace would automatically reflect in the containers. This is a convenient way for developing and verifying Hudi for developers who do not own a distributed environment. Note that this is how integration tests are run.

This helps avoid maintaining separate Docker images and avoids the costly step of building Hudi Docker images locally. But if users want to test Hudi from locations with lower network bandwidth, they can still build local images run the script `docker/build_local_docker_images.sh` to build local Docker images before running `docker/setup_demo.sh`

Here are the commands:

```
cd docker./build_local_docker_images.sh.....[INFO] Reactor Summary:[INFO][INFO] Hudi ............................................... SUCCESS [  2.507 s][INFO] hudi-common ........................................ SUCCESS [ 15.181 s][INFO] hudi-aws ........................................... SUCCESS [  2.621 s][INFO] hudi-timeline-service .............................. SUCCESS [  1.811 s][INFO] hudi-client ........................................ SUCCESS [  0.065 s][INFO] hudi-client-common ................................. SUCCESS [  8.308 s][INFO] hudi-hadoop-mr ..................................... SUCCESS [  3.733 s][INFO] hudi-spark-client .................................. SUCCESS [ 18.567 s][INFO] hudi-sync-common ................................... SUCCESS [  0.794 s][INFO] hudi-hive-sync ..................................... SUCCESS [  3.691 s][INFO] hudi-spark-datasource .............................. SUCCESS [  0.121 s][INFO] hudi-spark-common_2.11 ............................. SUCCESS [ 12.979 s][INFO] hudi-spark2_2.11 ................................... SUCCESS [ 12.516 s][INFO] hudi-spark_2.11 .................................... SUCCESS [ 35.649 s][INFO] hudi-utilities_2.11 ................................ SUCCESS [  5.881 s][INFO] hudi-utilities-bundle_2.11 ......................... SUCCESS [ 12.661 s][INFO] hudi-cli ........................................... SUCCESS [ 19.858 s][INFO] hudi-java-client ................................... SUCCESS [  3.221 s][INFO] hudi-flink-client .................................. SUCCESS [  5.731 s][INFO] hudi-spark3_2.12 ................................... SUCCESS [  8.627 s][INFO] hudi-dla-sync ...................................... SUCCESS [  1.459 s][INFO] hudi-sync .......................................... SUCCESS [  0.053 s][INFO] hudi-hadoop-mr-bundle .............................. SUCCESS [  5.652 s][INFO] hudi-hive-sync-bundle .............................. SUCCESS [  1.623 s][INFO] hudi-spark-bundle_2.11 ............................. SUCCESS [ 10.930 s][INFO] hudi-presto-bundle ................................. SUCCESS [  3.652 s][INFO] hudi-timeline-server-bundle ........................ SUCCESS [  4.804 s][INFO] hudi-trino-bundle .................................. SUCCESS [  5.991 s][INFO] hudi-hadoop-docker ................................. SUCCESS [  2.061 s][INFO] hudi-hadoop-base-docker ............................ SUCCESS [ 53.372 s][INFO] hudi-hadoop-base-java11-docker ..................... SUCCESS [ 48.545 s][INFO] hudi-hadoop-namenode-docker ........................ SUCCESS [  6.098 s][INFO] hudi-hadoop-datanode-docker ........................ SUCCESS [  4.825 s][INFO] hudi-hadoop-history-docker ......................... SUCCESS [  3.829 s][INFO] hudi-hadoop-hive-docker ............................ SUCCESS [ 52.660 s][INFO] hudi-hadoop-sparkbase-docker ....................... SUCCESS [01:02 min][INFO] hudi-hadoop-sparkmaster-docker ..................... SUCCESS [ 12.661 s][INFO] hudi-hadoop-sparkworker-docker ..................... SUCCESS [  4.350 s][INFO] hudi-hadoop-sparkadhoc-docker ...................... SUCCESS [ 59.083 s][INFO] hudi-hadoop-presto-docker .......................... SUCCESS [01:31 min][INFO] hudi-hadoop-trinobase-docker ....................... SUCCESS [02:40 min][INFO] hudi-hadoop-trinocoordinator-docker ................ SUCCESS [ 14.003 s][INFO] hudi-hadoop-trinoworker-docker ..................... SUCCESS [ 12.100 s][INFO] hudi-integ-test .................................... SUCCESS [ 13.581 s][INFO] hudi-integ-test-bundle ............................. SUCCESS [ 27.212 s][INFO] hudi-examples ...................................... SUCCESS [  8.090 s][INFO] hudi-flink_2.11 .................................... SUCCESS [  4.217 s][INFO] hudi-kafka-connect ................................. SUCCESS [  2.966 s][INFO] hudi-flink-bundle_2.11 ............................. SUCCESS [ 11.155 s][INFO] hudi-kafka-connect-bundle .......................... SUCCESS [ 12.369 s][INFO] ------------------------------------------------------------------------[INFO] BUILD SUCCESS[INFO] ------------------------------------------------------------------------[INFO] Total time:  14:35 min[INFO] Finished at: 2022-01-12T18:41:27-08:00[INFO] ------------------------------------------------------------------------
```

中文版
通过官方提供的样例我们可以构建docker镜像前提是已经安装了docker 和docker-compose

bash

```
# 下载docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# 查看docker-compose版本
docker-compose --version
```

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E7%BC%96%E8%AF%91Hudi%E6%BA%90%E7%A0%81 "编译Hudi源码")编译Hudi源码

修改maven的镜像源为阿里云的镜像

xml

```
<localRepository>/opt/module/apache-maven/repository</localRepository>
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
      <id>aliyunCentralMaven</id>
      <name>aliyun central maven</name>
      <url>https://maven.aliyun.com/repository/central/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
    <mirror>
      <id>centralMaven</id>
      <name>central maven</name>
      <url>http://mvnrepository.com/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

在hudi的项目路径下执行编译命令，编译也使用默认版本即可，如果为0.11版本之后则需要添加

shell

```
mvn clean package -Pintegration-tests -DskipTests
```

编译成功

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/20220722101020.png)

打包完成后执行setup\_demo.sh 拉取docker镜像。

![](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220722110548226.png)

docker ps 查看对应的进程

![image-20220808100120616](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808100120616.png)

**image-20220808100120616**

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E9%85%8D%E7%BD%AEHost%E6%98%A0%E5%B0%84 "配置Host映射")配置Host映射

xml

```
127.0.0.1 adhoc-1
127.0.0.1 adhoc-2
127.0.0.1 namenode
127.0.0.1 datanode1
127.0.0.1 hiveserver
127.0.0.1 hivemetastore
127.0.0.1 kafkabroker
127.0.0.1 sparkmaster
127.0.0.1 zookeeper
118.195.224.194 hudi
```

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%B8%8A%E4%BC%A0%E7%AC%AC%E4%B8%80%E6%89%B9%E6%95%B0%E6%8D%AE%E5%88%B0kafka "上传第一批数据到kafka")上传第一批数据到kafka

首先 brew install kcat 安装 kcat对数据进行加工处理。

shell

```
 cat docker/demo/data/batch_1.json | kcat -b kafkabroker -t stock_ticks -P
```

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E6%9F%A5%E7%9C%8BKafka%E6%95%B0%E6%8D%AE "查看Kafka数据")查看Kafka数据

使用命令查看对应的topic数据

shell

```
 kcat -C -b kafkabroker -t stock_ticks
```

![image-20220808100919416](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808100919416.png)

**image-20220808100919416**

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#Kafka%E4%B8%AD%E6%95%B0%E6%8D%AE%E5%86%99%E5%85%A5Hudi%E4%B8%AD "Kafka中数据写入Hudi中")Kafka中数据写入Hudi中

shell

```
docker exec -it adhoc-2 /bin/bash
# 使用下面的命令执行Delta-Looper并在HDFS中摄取到Stock_Ticks_cow表中
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type COPY_ON_WRITE \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts  \
  --target-base-path /user/hive/warehouse/stock_ticks_cow \
  --target-table stock_ticks_cow --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider
```

![image-20220808114709211](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114709211.png)

**image-20220808114709211**

![image-20220808101156661](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114741272.png)

**image-20220808101156661**

shell

```
 docker exec -it adhoc-2 /bin/bash
# 运行以下spark-subment命令以执行Delta-Fleoms，并在HDFS中摄取到Stock_Ticks_mor表
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type MERGE_ON_READ \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_mor \
  --target-table stock_ticks_mor \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
  --disable-compaction
```

![image-20220808101248582](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808114840170.png)

**image-20220808101248582**

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#hudi%E6%95%B0%E6%8D%AE%E4%B8%8Ehive%E9%9B%86%E6%88%90 "hudi数据与hive集成")hudi数据与hive集成

shell

```
docker exec -it adhoc-2 /bin/bash

/var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \
  --jdbc-url jdbc:hive2://hiveserver:10000 \
  --partitioned-by dt \
  --base-path /user/hive/warehouse/stock_ticks_cow \
  --database default \
  --table stock_ticks_cow
  

 /var/hoodie/ws/hudi-sync/hudi-hive-sync/run_sync_tool.sh \
  --jdbc-url jdbc:hive2://hiveserver:10000 \
  --user hive \
  --pass hive \
  --partitioned-by dt \
  --base-path /user/hive/warehouse/stock_ticks_mor \
  --database default \
  --table stock_ticks_mor 
```

对于hive表 stock\_ticks\_cow 支持快照查询和增量查询。

生成两张新表 stock\_ticks\_mor\_rt 和 stock\_ticks\_mor\_ro，前者支持Snapshot查询和Incremental查询，提供近实时的数据，后者支持读优化查询。

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Hive%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2 "使用Hive执行查询")使用Hive执行查询

运行 hive 查询以查找为股票代码“GOOG”提取的最新时间戳。您会注意到快照（对于 COW 和 MOR \_rt 表）和读取优化查询（对于 MOR \_ro 表）都给出了相同的值“上午 10:29”，因为 Hudi 为第一批数据创建了 parquet 文件。

shell

```
docker exec -it adhoc-2 /bin/bash

beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false
```

sql

```
root@adhoc-2:/opt# beeline -u jdbc:hive2://hiveserver:10000 \
>   --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
>   --hiveconf hive.stats.autogather=false
Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
# List Tables
0: jdbc:hive2://hiveserver:10000>  show tables;
+---------------------+--+
|      tab_name       |
+---------------------+--+
| stock_ticks_cow     |
| stock_ticks_mor_ro  |
| stock_ticks_mor_rt  |
+---------------------+--+
3 rows selected (0.3 seconds)

# Look at partitions that were added
0: jdbc:hive2://hiveserver:10000> show partitions stock_ticks_mor_rt;
+----------------+--+
|   partition    |
+----------------+--+
| dt=2018-08-31  |
+----------------+--+
1 row selected (0.16 seconds)

# COPY-ON-WRITE Queries:
=========================
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (7.834 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034650860    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034650860    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.747 seconds)
# Merge-On-Read Queries:
==========================

Lets run similar queries against M-O-R table. Lets look at both 
ReadOptimized and Snapshot(realtime data) queries supported by M-O-R table

# Run ReadOptimized Query. Notice that the latest timestamp is 10:29
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.19 seconds)
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.106 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.607 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Spark-%E6%9F%A5%E8%AF%A2 "使用Spark 查询")使用Spark 查询

scala

```
docker exec -it adhoc-1 /bin/bash

$SPARK_INSTALL/bin/spark-shell \
  --jars $HUDI_SPARK_BUNDLE \
  --master local[2] \
  --driver-class-path $HADOOP_CONF_DIR \
  --conf spark.sql.hive.convertMetastoreParquet=false \
  --deploy-mode client \
  --driver-memory 1G \
  --executor-memory 3G \
  --num-executors 1


23/11/03 01:14:38 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1698974085962).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.4.1
      /_/

Using Scala version 2.12.17 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.



scala> spark.sql("show tables").show(100, false)
23/11/03 01:15:15 WARN HiveConf: HiveConf of name hive.metastore.event.db.notification.api.auth does not exist
+---------+------------------+-----------+
|namespace|tableName         |isTemporary|
+---------+------------------+-----------+
|default  |stock_ticks_cow   |false      |
|default  |stock_ticks_mor_ro|false      |
|default  |stock_ticks_mor_rt|false      |
+---------+------------------+-----------+


# Copy-On-Write Table

## Run max timestamp query against COW table

scala> spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
23/11/03 01:15:39 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
23/11/03 01:15:39 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

## Projection Query

scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20231102122758903  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20231102122758903  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+


# Merge-On-Read Queries:
==========================

Lets run similar queries against M-O-R table. Lets look at both
ReadOptimized and Snapshot queries supported by M-O-R table

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

# Run Snapshot Query. Notice that the latest timestamp is again 10:29

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+


# Run Read Optimized and Snapshot project queries

scala>  spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+



scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Presto%E6%9F%A5%E8%AF%A2 "使用Presto查询")使用Presto查询

shell

```
docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090


[root@biodata bin]# docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090

presto> show catalogs;
  Catalog
-----------
 hive
 jmx
 localfile
 system
(4 rows)

Query 20231103_011713_00000_hbfgy, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
0:04 [0 rows, 0B] [0 rows/s, 0B/s]


presto>  use hive.default;
USE
presto:default>  show tables;
       Table
--------------------
 stock_ticks_cow
 stock_ticks_mor_ro
 stock_ticks_mor_rt
(3 rows)

Query 20231103_011755_00002_hbfgy, FINISHED, 2 nodes
Splits: 19 total, 19 done (100.00%)
0:02 [3 rows, 102B] [1 rows/s, 60B/s]


presto:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1
--------+---------------------
 GOOG   | 2018-08-31 10:29:00
(1 row)

Query 20231103_011815_00003_hbfgy, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:08 [197 rows, 426KB] [25 rows/s, 56KB/s]



presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close
---------------------+--------+---------------------+--------+-----------+----------
 20231102122758903   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02
 20231102122758903   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085
(2 rows)

Query 20231103_011836_00004_hbfgy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
404ms [197 rows, 429KB] [487 rows/s, 1.04MB/s]


presto:default>  select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1
--------+---------------------
 GOOG   | 2018-08-31 10:29:00
(1 row)

Query 20231103_011901_00005_hbfgy, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
324ms [197 rows, 426KB] [608 rows/s, 1.29MB/s]


presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close
---------------------+--------+---------------------+--------+-----------+----------
 20231102123006819   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02
 20231102123006819   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085
(2 rows)

Query 20231103_011919_00006_hbfgy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
255ms [197 rows, 429KB] [773 rows/s, 1.65MB/s]

```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Trino%E6%9F%A5%E8%AF%A2 "使用Trino查询")使用Trino查询

这是使用Trino的查询，目前Trino不支持快照和增量查询在hudi表上

shell

```
docker exec -it adhoc-2 trino --server trino-coordinator-1:8091
trino>  show catalogs;
 Catalog
---------
 hive
 system
(2 rows)

Query 20231103_012301_00000_yy9yn, FINISHED, 1 node
Splits: 11 total, 11 done (100.00%)
3.51 [0 rows, 0B] [0 rows/s, 0B/s]


trino>  use hive.default;
USE
trino:default> show tables;
       Table
--------------------
 stock_ticks_cow
 stock_ticks_mor_ro
 stock_ticks_mor_rt
(3 rows)

Query 20231103_012325_00004_yy9yn, FINISHED, 2 nodes
Splits: 11 total, 11 done (100.00%)
0.91 [3 rows, 102B] [3 rows/s, 112B/s]

trino:default>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
  symbol |        _col1
--------+---------------------
 GOOG   | 2018-08-31 10:29:00
(1 row)

Query 20231103_012344_00005_yy9yn, FINISHED, 1 node
Splits: 25 total, 25 done (100.00%)
7.06 [197 rows, 442KB] [27 rows/s, 62.6KB/s]


trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close from stock_ticks_cow where symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close
---------------------+--------+---------------------+--------+-----------+----------
 20231102122758903   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02
 20231102122758903   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085
(2 rows)

Query 20231103_012404_00006_yy9yn, FINISHED, 1 node
Splits: 9 total, 9 done (100.00%)
0.27 [197 rows, 450KB] [729 rows/s, 1.63MB/s]


trino:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1
--------+---------------------
 GOOG   | 2018-08-31 10:29:00
(1 row)

Query 20231103_012419_00007_yy9yn, FINISHED, 1 node
Splits: 25 total, 25 done (100.00%)
0.40 [197 rows, 442KB] [498 rows/s, 1.09MB/s]


trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_035928_00008_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.46 [197 rows, 450KB] [428 rows/s, 978KB/s]
```

### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%B8%8A%E4%BC%A0%E7%AC%AC%E4%BA%8C%E6%89%B9%E6%95%B0%E6%8D%AE "上传第二批数据")上传第二批数据

使用spark delta-streamer 上传这批数据，这批数据没有产生新的分区不需要使用ihve-sync来同步数据

shell

```
cat docker/demo/data/batch_2.json | kcat -b kafkabroker -t stock_ticks -P
docker exec -it adhoc-2 /bin/bash
```

使用spark-submit delta-streamer 提取数据到HDFS上的stock\_ticks\_cow

shell

```
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type COPY_ON_WRITE \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_cow \
  --target-table stock_ticks_cow \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider
```

使用spark-submit delta-streamer 提取数据到HDFS上的stock\_ticks\_mor

scala

```
spark-submit \
  --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer $HUDI_UTILITIES_BUNDLE \
  --table-type MERGE_ON_READ \
  --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
  --source-ordering-field ts \
  --target-base-path /user/hive/warehouse/stock_ticks_mor \
  --target-table stock_ticks_mor \
  --props /var/demo/config/kafka-source.properties \
  --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
  --disable-compaction
```

生成的数据 cow表中生成新版本的parquet文件

Merge-On-Read table表中，第二批新增的数据生成了尚未合并的delta(log)文件

![image-20220808133212806](https://hphimages-1253879422.cos.ap-beijing.myqcloud.com/%E6%95%B0%E6%8D%AE%E6%B9%96/image-20220808133212806.png)

**image-20220808133212806**

使用 Copy-On-Write 表，一旦批次提交，快照查询会立即将更改视为第二批的一部分，因为每次摄取都会创建更新版本的 parquet 文件。

使用 Merge-On-Read 表，第二次摄取只是将批处理附加到未合并的增量（日志）文件中。这是 ReadOptimized 和 Snapshot 查询将提供不同结果的时候。ReadOptimized 查询仍将返回“10:29 am”，因为它只会从 Parquet 文件中读取。快照查询将进行即时合并并返回最新提交的数据，即“上午 10:59”。

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8HIve%E6%9F%A5%E8%AF%A2 "使用HIve查询")使用HIve查询

shell

```
docker exec -it adhoc-2 /bin/bash
beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false

Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
# Copy On Write Table:
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (2.217 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220725021154599    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220725022945766    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.659 seconds)
As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.

# Merge On Read Table:
# Read Optimized Query

0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.055 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.692 seconds)
# Snapshot Query

0: jdbc:hive2://hiveserver:10000>  select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (1.978 seconds)
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.599 seconds)
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Spark%E6%9F%A5%E8%AF%A2 "使用Spark查询")使用Spark查询

scala

```
root@adhoc-1:/opt# $SPARK_INSTALL/bin/spark-shell \
>   --jars $HUDI_SPARK_BUNDLE \
>   --driver-class-path $HADOOP_CONF_DIR \
>   --conf spark.sql.hive.convertMetastoreParquet=false \
>   --deploy-mode client \
>   --driver-memory 1G \
>   --master local[2] \
>   --executor-memory 3G \
>   --num-executors 1
22/07/25 03:03:57 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1658718244614).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/
         
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.

// Copy On Write Table:
scala>  spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+                                                    
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+



// As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.
// Merge On Read Table:
// Read Optimized Query
scala>  spark.sql("select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+


scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034650860  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040120355  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
As you can notice, the above queries now reflect the changes that came as part of ingesting second batch.
// Merge On Read Table:
// Read Optimized Query

scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:29:00|
+------+-------------------+

scala>  spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808034809720  |GOOG  |2018-08-31 10:29:00|3391  |1230.1899|1230.085|
+-------------------+------+-------------------+------+---------+--------+
// Snapshot Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+

scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Presto%E6%9F%A5%E8%AF%A2-1 "使用Presto查询")使用Presto查询

运行同样的查询语句在ReadOptimized上

sql

```
root@biodata ~]# docker exec -it presto-worker-1 presto --server presto-coordinator-1:8090

presto> use hive.default;
USE
presto:default> select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_054014_00011_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [225 rows/s, 488KB/s]

presto:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040120355   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_054026_00012_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
484ms [197 rows, 429KB] [406 rows/s, 885KB/s]

presto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_054038_00013_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [316 rows/s, 685KB/s]

presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_054043_00014_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
450ms [197 rows, 429KB] [438 rows/s, 954KB/s]
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Trino%E6%9F%A5%E8%AF%A2-1 "使用Trino查询")使用Trino查询

sql

```
root@biodata ~]# docker exec -it adhoc-2 trino --server trino-coordinator-1:8091

trino>  use hive.default;
USE
trino:default>  select symbol, max(ts) from stock_ticks_cow group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_054140_00012_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
0.47 [197 rows, 442KB] [423 rows/s, 951KB/s]

trino:default> select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034650860   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040120355   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_054150_00013_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.38 [197 rows, 450KB] [517 rows/s, 1.15MB/s]

trino:default>  select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:29:00 
(1 row)

Query 20220808_054156_00014_vxj2r, FINISHED, 1 node
Splits: 13 total, 13 done (100.00%)
0.43 [197 rows, 442KB] [460 rows/s, 1.01MB/s]

trino:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808034809720   | GOOG   | 2018-08-31 10:29:00 |   3391 | 1230.1899 | 1230.085 
(2 rows)

Query 20220808_054201_00015_vxj2r, FINISHED, 1 node
Splits: 5 total, 5 done (100.00%)
0.38 [197 rows, 450KB] [523 rows/s, 1.17MB/s]
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E5%A2%9E%E9%87%8F%E6%9F%A5%E8%AF%A2COW%E8%A1%A8 "增量查询COW表")增量查询COW表

shell

```
docker exec -it adhoc-2 /bin/bash
beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false

Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000>  select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034650860    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040120355    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.606 seconds)
```

上面的查询查到了条cimmit记录一个是20220808034650860，另一个是20220808040120355 在时间线上。

为了展示增量查询的效果，我们假设读者已经将这些更改视为摄取第一批的一部分。现在，为了让读者看到第二批的效果，他/她必须将开始时间戳保持到第一批的提交时间（20180924064621）并运行增量查询

Hudi 增量模式通过使用 hudi 管理的元数据过滤掉没有任何候选行的文件，为增量查询提供高效扫描。使用上述设置，从提交 20220808034650860 中没有任何更新的文件 ID 将被过滤掉而不进行扫描。这是增量查询：

sql

```
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow where  symbol = 'GOOG' and `_hoodie_commit_time` > '20220808034650860';
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Spark-SQL%E8%BF%9B%E8%A1%8C%E5%A2%9E%E9%87%8F%E6%9F%A5%E8%AF%A2 "使用Spark SQL进行增量查询")使用Spark SQL进行增量查询

scala

```
docker exec -it adhoc-1 /bin/bash
root@adhoc-1:/opt# 
$SPARK_INSTALL/bin/spark-shell \
   --jars $HUDI_SPARK_BUNDLE \
   --driver-class-path $HADOOP_CONF_DIR \
   --conf spark.sql.hive.convertMetastoreParquet=false \
   --deploy-mode client \
   --driver-memory 1G \
   --master local[2] \
   --executor-memory 3G \
   --num-executors 1

22/07/25 05:41:53 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://adhoc-1:4040
Spark context available as 'sc' (master = local[2], app id = local-1658727720839).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/
         
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_212)
Type in expressions to have them evaluated.
Type :help for more information.


import org.apache.hudi.DataSourceReadOptions


val hoodieIncViewDF =  spark.read.format("org.apache.hudi").option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY, DataSourceReadOptions.QUERY_TYPE_INCREMENTAL_OPT_VAL).option(DataSourceReadOptions.BEGIN_INSTANTTIME_OPT_KEY, "20220808034650860").load("/user/hive/warehouse/stock_ticks_cow")



hoodieIncViewDF.registerTempTable("stock_ticks_cow_incr_tmp1")
spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_cow_incr_tmp1 where  symbol = 'GOOG'").show(100, false);
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E6%89%8B%E5%B7%A5%E5%AE%8C%E6%88%90%E5%90%88%E5%B9%B6%E6%96%87%E4%BB%B6 "手工完成合并文件")手工完成合并文件

sql

```
docker exec -it adhoc-1 /bin/bash
/var/hoodie/ws/hudi-cli/hudi-cli.sh

===================================================================
*         ___                          ___                        *
*        /\__\          ___           /\  \           ___         *
*       / /  /         /\__\         /  \  \         /\  \        *
*      / /__/         / /  /        / /\ \  \        \ \  \       *
*     /  \  \ ___    / /  /        / /  \ \__\       /  \__\      *
*    / /\ \  /\__\  / /__/  ___   / /__/ \ |__|     / /\/__/      *
*    \/  \ \/ /  /  \ \  \ /\__\  \ \  \ / /  /  /\/ /  /         *
*         \  /  /    \ \  / /  /   \ \  / /  /   \  /__/          *
*         / /  /      \ \/ /  /     \ \/ /  /     \ \__\          *
*        / /  /        \  /  /       \  /  /       \/__/          *
*        \/__/          \/__/         \/__/    Apache Hudi CLI    *
*                                                                 *
===================================================================

Welcome to Apache Hudi CLI. Please type help if you are looking for help. 
hudi->connect --path /user/hive/warehouse/stock_ticks_mor
Metadata for table stock_ticks_mor loaded
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compaction schedule --hoodieConfigs hoodie.compact.inline.max.delta.commits=1
22/08/08 06:19:09 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/08/08 06:19:11 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:19:11 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
22/08/08 06:19:19 WARN HoodieCompactor: After filtering, Nothing to compact for /user/hive/warehouse/stock_ticks_mor
Attempted to schedule compaction for 20220808061907889
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

hudi:stock_ticks_mor->
hudi:stock_ticks_mor->compaction schedule --hoodieConfigs hoodie.compact.inline.max.delta.commits=1
22/08/08 06:19:34 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/08/08 06:19:36 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:19:36 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
22/08/08 06:19:44 WARN HoodieCompactor: After filtering, Nothing to compact for /user/hive/warehouse/stock_ticks_mor
Attempted to schedule compaction for 20220808061933685
hudi:stock_ticks_mor->
hudi:stock_ticks_mor->refresh
Metadata for table stock_ticks_mor refreshed.
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ REQUESTED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝

# Schedule a compaction. This will use Spark Launcher to schedule compaction
hudi:stock_ticks_mor->compaction run --compactionInstant  20220808055239150 --parallelism 2 --sparkMemory 1G  --schemaFilePath /var/demo/config/schema.avsc --retry 1  
22/08/08 06:20:06 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

22/08/08 06:20:08 WARN DFSPropertiesConfiguration: Cannot find HUDI_CONF_DIR, please set it as the dir of hudi-defaults.conf
22/08/08 06:20:08 WARN DFSPropertiesConfiguration: Properties file file:/etc/hudi/conf/hudi-defaults.conf not found. Ignoring to load props file
00:04  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
00:05  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
00:07  WARN: Timeline-server-based markers are not supported for HDFS: base path /user/hive/warehouse/stock_ticks_mor.  Falling back to direct markers.
Compaction successfully completed for 20220808055239150
# Now refresh and check again. You will see that there is a new compaction requested
hudi:stock_ticks_mor->refresh
Metadata for table stock_ticks_mor refreshed.
hudi:stock_ticks_mor->compactions show all
╔═════════════════════════╤═══════════╤═══════════════════════════════╗
║ Compaction Instant Time │ State     │ Total FileIds to be Compacted ║
╠═════════════════════════╪═══════════╪═══════════════════════════════╣
║ 20220808055239150       │ COMPLETED │ 1                             ║
╚═════════════════════════╧═══════════╧═══════════════════════════════╝
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E4%BD%BF%E7%94%A8Hive%E8%BF%9B%E8%A1%8C%E5%A2%9E%E9%87%8F%E6%9F%A5%E8%AF%A2 "使用Hive进行增量查询")使用Hive进行增量查询

sql

```
docker exec -it adhoc-2 /bin/bash

beeline -u jdbc:hive2://hiveserver:10000 \
  --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
  --hiveconf hive.stats.autogather=false
  
# Read Optimized Query
root@adhoc-2:/opt# beeline -u jdbc:hive2://hiveserver:10000 \
>   --hiveconf hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat \
>   --hiveconf hive.stats.autogather=false
Connecting to jdbc:hive2://hiveserver:10000
Connected to: Apache Hive (version 2.3.3)
Driver: Hive JDBC (version 1.2.1.spark2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1.spark2 by Apache Hive
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:29:00  |
+---------+----------------------+--+
1 row selected (2.097 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808034809720    | GOOG    | 2018-08-31 10:29:00  | 3391    | 1230.1899  | 1230.085  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.707 seconds)

# Snapshot Query
0: jdbc:hive2://hiveserver:10000> select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG';
WARNING: Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
+---------+----------------------+--+
| symbol  |         _c1          |
+---------+----------------------+--+
| GOOG    | 2018-08-31 10:59:00  |
+---------+----------------------+--+
1 row selected (1.967 seconds)
0: jdbc:hive2://hiveserver:10000> select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808034809720    | GOOG    | 2018-08-31 09:59:00  | 6330    | 1230.5     | 1230.02   |
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
2 rows selected (0.569 seconds)
# Incremental Query:
set hoodie.stock_ticks_mor.consume.mode=INCREMENTAL;
set hoodie.stock_ticks_mor.consume.max.commits=3;
set hoodie.stock_ticks_mor.consume.start.timestamp=20220808034809720;

select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG' and `_hoodie_commit_time` > '20220808034809720';
+----------------------+---------+----------------------+---------+------------+-----------+--+
| _hoodie_commit_time  | symbol  |          ts          | volume  |    open    |   close   |
+----------------------+---------+----------------------+---------+------------+-----------+--+
| 20220808040149926    | GOOG    | 2018-08-31 10:59:00  | 9021    | 1227.1993  | 1227.215  |
+----------------------+---------+----------------------+---------+------------+-----------+--+
  
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#%E8%AF%BB%E4%BC%98%E5%8C%96%E5%92%8C%E5%BF%AB%E7%85%A7%E6%9F%A5%E8%AF%A2-MOR%E4%BD%BF%E7%94%A8Spark "读优化和快照查询 MOR使用Spark")读优化和快照查询 MOR使用Spark

scala

```
docker exec -it adhoc-1 /bin/bash
$SPARK_INSTALL/bin/spark-shell \
  --jars $HUDI_SPARK_BUNDLE \
  --driver-class-path $HADOOP_CONF_DIR \
  --conf spark.sql.hive.convertMetastoreParquet=false \
  --deploy-mode client \
  --driver-memory 1G \
  --master local[2] \
  --executor-memory 3G \
  --num-executors 1

# Read Optimized Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+                                                    
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+
scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
# Snapshot Query
scala> spark.sql("select symbol, max(ts) from stock_ticks_mor_rt group by symbol HAVING symbol = 'GOOG'").show(100, false)
+------+-------------------+
|symbol|max(ts)            |
+------+-------------------+
|GOOG  |2018-08-31 10:59:00|
+------+-------------------+
scala> spark.sql("select `_hoodie_commit_time`, symbol, ts, volume, open, close  from stock_ticks_mor_rt where  symbol = 'GOOG'").show(100, false)
+-------------------+------+-------------------+------+---------+--------+
|_hoodie_commit_time|symbol|ts                 |volume|open     |close   |
+-------------------+------+-------------------+------+---------+--------+
|20220808034809720  |GOOG  |2018-08-31 09:59:00|6330  |1230.5   |1230.02 |
|20220808040149926  |GOOG  |2018-08-31 10:59:00|9021  |1227.1993|1227.215|
+-------------------+------+-------------------+------+---------+--------+
```

#### [](http://hphblog.cn/2022/08/06/apache-hudi-kuai-su-ti-yan/#Presto-%E8%AF%BB%E4%BC%98%E5%8C%96%E6%9F%A5%E8%AF%A2MOR%E8%A1%A8 "Presto 读优化查询MOR表")Presto 读优化查询MOR表

sql

```



presto> use hive.default;
USE

# Read Optimized Query
presto:default> select symbol, max(ts) from stock_ticks_mor_ro group by symbol HAVING symbol = 'GOOG';
 symbol |        _col1        
--------+---------------------
 GOOG   | 2018-08-31 10:59:00 
(1 row)

Query 20220808_064105_00016_hq6qp, FINISHED, 1 node
Splits: 49 total, 49 done (100.00%)
0:01 [197 rows, 426KB] [300 rows/s, 651KB/s]

presto:default>  select "_hoodie_commit_time", symbol, ts, volume, open, close  from stock_ticks_mor_ro where  symbol = 'GOOG';
 _hoodie_commit_time | symbol |         ts          | volume |   open    |  close   
---------------------+--------+---------------------+--------+-----------+----------
 20220808034809720   | GOOG   | 2018-08-31 09:59:00 |   6330 |    1230.5 |  1230.02 
 20220808040149926   | GOOG   | 2018-08-31 10:59:00 |   9021 | 1227.1993 | 1227.215 
(2 rows)

Query 20220808_064117_00017_hq6qp, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
469ms [197 rows, 429KB] [420 rows/s, 915KB/s]
```

编译镜像命令
``
mvn clean pre-integration-test -DskipTests -Ddocker.compose.skip=true -Ddocker.build.skip=false 
``
hive代码重编译execlude
```
<!--                  <include>com.google.guava:guava</include>-->
<!--                  <include>org.apache.parquet:parquet-hadoop-bundle</include>-->


 mvn clean package -Pdist -DskipTests
```

### - 1 修改pom 版本为2.3.4（根据自己项目引用版本）
 
 - 2 根pom注释掉多余不需要打包的，修改hive-exec的artifactId（因为后续要上传公司私服，避免与hive-exec本身冲突），
 
 - 3 include项删掉guava，见上面的图

 - 4 打包hive(根目录)：  mvn clean package -Pdist -DskipTests （注意：打包hive-common在window会报错bash找不到，我是在linux上打包的）

 - 5 在ql包下ql/target/拿到hive-exec的包

 - 6 上传修改过artifactId的jar包到公司私服（打包修改为你公司私服地址和版本等）比如：
  mvn deploy:deploy-file -DgroupId=org.apache.hive -DartifactId=hive-exec-xxx -Dversion=2.3.4-SNAPSHOT -Dpackaging=jar  -Durl=http://repo.xxxx.com/repositories/snapshots -DrepositoryId=xxxx-snapshot -Dfile=hive-exec-xxx-2.3.4.jar

 - 7 修改你的项目依赖为自己新上传私服的hive-exec-xxx

 - 8 重新打包引用项目，冲突问题解决~（注：如果hive-exec出现非guava冲突，比如：commons-lang commons-lang3的冲突也是一样的解决方式，原因都是hive-exec include了这些依赖，即打包到hive中了，所以会exclusion不掉）
