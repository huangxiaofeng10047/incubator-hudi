
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.

ARG HADOOP_VERSION=3.2.1
ARG HIVE_VERSION=3.1.3
ARG SPARK_VERSION=3.4.1
FROM apachehudi/hudi-hadoop_${HADOOP_VERSION}-base:1.0.0-SNAPSHOT

ENV HIVE_HOME /opt/hive
ENV PATH $HIVE_HOME/bin:$PATH
ENV HADOOP_HOME /opt/hadoop-$HADOOP_VERSION

WORKDIR /opt

ARG HADOOP_VERSION=3.2.1
ARG HIVE_VERSION=3.1.3
ARG SPARK_VERSION=3.4.1
ARG HIVE_URL=https://archive.apache.org/dist/hive/hive-$HIVE_VERSION/apache-hive-$HIVE_VERSION-bin.tar.gz
ENV HIVE_VERSION ${HIVE_VERSION}
ENV HIVE_URL ${HIVE_URL}

#Install Hive MySQL, PostgreSQL JDBC
RUN echo "Hive URL is :${HIVE_URL}" && wget ${HIVE_URL} -O hive.tar.gz && \
	tar -xzvf hive.tar.gz && mv *hive*-bin hive && \
        ln -s /usr/share/java/mysql-connector-java.jar $HIVE_HOME/lib/mysql-connector-java.jar && \
	wget https://jdbc.postgresql.org/download/postgresql-9.4.1212.jar -O $HIVE_HOME/lib/postgresql-jdbc.jar && \
	rm hive.tar.gz && mkdir -p /var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/

#Spark should be compiled with Hive to be able to use it
#hive-site.xml should be copied to $SPARK_HOME/conf folder

#Custom configuration goes here
ADD conf/hive-site.xml $HADOOP_CONF_DIR
ADD conf/beeline-log4j2.properties $HIVE_HOME/conf
ADD conf/hive-env.sh $HIVE_HOME/conf
ADD conf/hive-exec-log4j2.properties $HIVE_HOME/conf
ADD conf/hive-log4j2.properties $HIVE_HOME/conf
ADD conf/ivysettings.xml $HIVE_HOME/conf
ADD conf/llap-daemon-log4j2.properties $HIVE_HOME/conf

# Setup Hoodie Library jars
ADD target/ /var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/

ENV HUDI_HADOOP_BUNDLE=/var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/hoodie-hadoop-mr-bundle.jar
ENV HUDI_HIVE_SYNC_BUNDLE=/var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/hoodie-hive-sync-bundle.jar
ENV HUDI_SPARK_BUNDLE=/var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/hoodie-spark-bundle.jar
ENV HUDI_UTILITIES_BUNDLE=/var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/hoodie-utilities.jar
ENV HUDI_INTEG_TEST_BUNDLE=/var/hoodie/ws/docker/hoodie/hadoop/hive_base/target/hoodie-integ-test-bundle.jar

COPY startup.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/startup.sh

COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh
#//$ rm /opt/shared/apache-hive-3.1.2-bin/lib/guava-19.0.jar
#  $ cp /opt/shared/hadoop-3.2.1/share/hadoop/hdfs/lib/guava-27.0-jre.jar /opt/shared/apache-hive-3.1.2-bin/lib/
RUN rm $HIVE_HOME/lib/guava-19.0.jar && cp $HADOOP_HOME/share/hadoop/hdfs/lib/guava-27.0-jre.jar $HIVE_HOME/lib/
RUN rm $HIVE_HOME/lib/hive-exec-3.1.3.jar &&  \
    wget -nv -O $HIVE_HOME/lib/hive-exec-3.1.3.jar http://10.7.20.31:60000/hive-exec-3.1.3.jar
ENV PATH $HIVE_HOME/bin/:$PATH
# NOTE: This is the only battle-proven method to inject jars into Hive CLI
ENV AUX_CLASSPATH=file://${HUDI_HADOOP_BUNDLE}
EXPOSE 10000/tcp
EXPOSE 10002/tcp

ENTRYPOINT ["entrypoint.sh"]
CMD startup.sh
