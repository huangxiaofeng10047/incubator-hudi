在hive的beeline上设置
set mapred.child.java.opts=-Xmx1536m -Xms1536m -Xmn256m -XX:SurvivorRatio=6 -XX:MaxPermSize=128m -XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=80  -XX:GCTimeLimit=90 
-XX:GCHeapFreeLimit=10 -XX:ParallelGCThreads=8
hive sql执行时报错了heap out of memory
解决办法，在hive-env.sh追加：
export HADOOP_HEAPSIZE=2048

set hive.vectorized.execution.enabled = false;
这个命令是解决这个问题的：
Caused by: java.lang.ClassCastException: org.apache.hadoop.hive.ql.exec.vector.VectorizedRowBatch cannot be cast to org.apache.hadoop.io.ArrayWritable
	at org.apache.hudi.hadoop.realtime.RealtimeCompactedRecordReader.createValue(RealtimeCompactedRecordReader.java:183)
	at org.apache.hudi.hadoop.realtime.RealtimeCompactedRecordReader.createValue(RealtimeCompactedRecordReader.java:47)
	at org.apache.hudi.hadoop.realtime.HoodieRealtimeRecordReader.createValue(HoodieRealtimeRecordReader.java:89)
	at org.apache.hudi.hadoop.realtime.HoodieRealtimeRecordReader.createValue(HoodieRealtimeRecordReader.java:36)
	at org.apache.hadoop.hive.ql.io.HiveRecordReader.createValue(HiveRecordReader.java:58)
	at org.apache.hadoop.hive.ql.io.HiveRecordReader.createValue(HiveRecordReader.java:33)
	at org.apache.hadoop.mapred.split.TezGroupedSplitsInputFormat$TezGroupedSplitsRecordReader.createValue(TezGroupedSplitsInputFormat.java:160)
	at org.apache.tez.mapreduce.lib.MRReaderMapred.setupOldRecordReader(MRReaderMapred.java:168)
	at org.apache.tez.mapreduce.lib.MRReaderMapred.setSplit(MRReaderMapred.java:83)
	at org.apache.tez.mapreduce.input.MRInput.initFromEventInternal(MRInput.java:706)
	at org.apache.tez.mapreduce.input.MRInput.initFromEvent(MRInput.java:665)
	at org.apache.tez.mapreduce.input.MRInputLegacy.checkAndAwaitRecordReaderInitialization(MRInputLegacy.java:150)
	at org.apache.tez.mapreduce.input.MRInputLegacy.init(MRInputLegacy.java:114)
	at org.apache.hadoop.hive.ql.exec.tez.MapRecordProcessor.getMRInput(MapRecordProcessor.java:525)
	at org.apache.hadoop.hive.ql.exec.tez.MapRecordProcessor.init(MapRecordProcessor.java:171)
	at org.apache.hadoop.hive.ql.exec.tez.TezProcessor.initializeAndRunProcessor(TezProcessor.java:266)



关闭hive的vector执行，会引起下面的问题。
    
rt table with pure log is not supported well for Hive queries, you may need to switch to ro table instead.

在Apache Hudi（Hadoop Upsert, Delete, and Incremental）中，"RO" 和 "RT" 表是 MOR（Merge on Read）模式的两个关键概念，它们代表着不同的数据表。

1. **RO 表（Read-Optimized 表）**：
   - RO 表是一个只读的表，它包含了数据的一系列快照（snapshots）。每次数据写入时，Hudi 会在 RO 表中创建一个新的数据文件，每个文件代表一个时间戳的数据快照。
   - RO 表非常适合读取操作，因为数据文件是不可变的，这意味着在读取时可以高效地进行并行处理，而无需担心并发写入的复杂性。
   - RO 表通常用于支持查询操作，如批量分析任务。由于只包含数据快照，不支持原子的插入、更新或删除操作。

2. **RT 表（Real-Time 表）**：
   - RT 表是一个可写的表，用于支持实时写入数据。在 MOR 模式下，当需要进行插入、更新或删除操作时，这些操作会被写入 RT 表。
   - RT 表支持类似数据库的事务性写入操作，它允许应用程序以原子方式插入新数据、更新现有数据和删除数据。
   - RT 表的目的是为了支持实时应用程序，可以在写入和读取之间保持低延迟，同时提供了写入操作的原子性和实时性。

总结：
- RO 表是只读的，用于支持高效的批量查询和分析操作，每个数据文件代表一个时间戳的数据快照。
- RT 表是可写的，用于支持实时写入和支持实时应用程序，它允许插入、更新和删除操作，并提供了写入操作的原子性和实时性。

通常，Hudi 在 MOR 模式下同时维护 RO 表和 RT 表，以满足不同类型的数据操作需求。 RO 表适用于批量查询和分析，而 RT 表适用于实时数据写入和查询。


Using Vectorized Query Execution
Enabling vectorized execution
To use vectorized query execution, you must store your data in ORC format, and set the following variable as shown in Hive SQL (see Configuring Hive):

set hive.vectorized.execution.enabled = true;

Vectorized execution is off by default, so your queries only utilize it if this variable is turned on. To disable vectorized execution and go back to standard execution, do the following:

set hive.vectorized.execution.enabled = false;

Additional configuration variables for vectorized execution are documented in Configuration Properties – Vectorization.


docker tag apachehudi/hudi-hadoop_3.2.1-hive_3.1.3-sparkmaster_3.4.1:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3-sparkmaster_3.4.1:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3-sparkmaster_3.4.1:1.0.0-SNAPSHOT
docker tag apachehudi/hudi-hadoop_3.2.1-hive_3.1.3:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3:1.0.0-SNAPSHOT
docker tag apachehudi/hudi-hadoop_3.2.1-datanode:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-datanode:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-datanode:1.0.0-SNAPSHOT
docker tag apachehudi/hudi-hadoop_3.2.1-namenode:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-namenode:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-namenode:1.0.0-SNAPSHOT

docker tag apachehudi/hudi-hadoop_3.2.1-history:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-history:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-history:1.0.0-SNAPSHOT
docker tag apachehudi/hudi-hadoop_3.2.1-hive_3.1.3-sparkworker_3.4.1:1.0.0-SNAPSHOT huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3-sparkworker_3.4.1:1.0.0-SNAPSHOT
docker push huangxiaofenglogin/hudi-hadoop_3.2.1-hive_3.1.3-sparkworker_3.4.1:1.0.0-SNAPSHOT