---
layout:     post
title:      "整合 Flume+Kafka+Spark 实战配置（macOS）"
subtitle:   "Flume+Kafka+Spark combat configuration (macOS or Linux)"
date:       2018-10-20
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-flume-02.jpg"
categories: [Big Data]
tags: Flume Kafka Spark
---
本文概览：
* TOC
{:toc}

> 本文档的所有设定 `IP` 地方的地方均以 `192.168.100.100` 为例，实际操作中请修改为自己的本机 `IP`

## 一、Flume
### 1.1 配置
首先下载 `Flume`，下载地址：[http://www.apache.org/dyn/closer.lua/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz](http://www.apache.org/dyn/closer.lua/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz)

解压至 `~/app/`，然后配置环境变量，在相应的文件（`MacOS`是`.zshrc`文件，`Linux`是`.bashrc`或`.profile`文件）中添加以下代码：

```properties
# set Flume
export FLUME_HOME=/Users/pengtuo/app/apache-flume-1.8.0-bin
export PATH=$FLUME_HOME/bin:$PATH
```

其中的路径改成自己的 `Flume` 文件路径。

> 是否配置环境变量可以依个人心情，配置环境变量的好处是可以在任何目录下启动 `Flume`。

配置好环境变量后，首先在 `Flume` 端进行级联配置，本次案例为两级`Flume`，其中每级 `Flume` 进行双 `channel` 与双 `sink` 配置，一个 `channel` 与 `sink` 指向下一级流动，另一个 `channel` 与 `sink` 则进行控制台输出作用，最后一层 `Flume` 指向 `Kafka`，两级配置都贴出来供参考：

第一级`Flume`的配置如下，配置文件名为`flume-first.properties`：

```properties
# The configuration file needs to define the sources, the channels and the sinks.
# Sources, channels and sinks are defined per agent

agent_http.sources = collection
agent_http.channels = ch1 ch2
agent_http.sinks = logger-sink avro-sink

agent_http.sources.collection.selector.type=replicating

# Properties of source
agent_http.sources.collection.channels = ch1 ch2
agent_http.sources.collection.type = http
agent_http.sources.collection.bind = 192.168.100.100
agent_http.sources.collection.port = 44444
agent_http.sources.collection.handler = com.pengtuo.handler.ExampleHandler

# Properties of channel ch1
agent_http.channels.ch1.type = memory
agent_http.channels.ch1.capacity  = 1000
agent_http.channels.ch1.transactionCapacity  = 100

# Properties of channel ch2
agent_http.channels.ch2.type = memory
agent_http.channels.ch2.capacity  = 1000
agent_http.channels.ch2.transactionCapacity  = 100

# Properties of sink avro-sink
agent_http.sinks.avro-sink.channel = ch1
agent_http.sinks.avro-sink.type = avro
agent_http.sinks.avro-sink.hostname = 192.168.100.100
agent_http.sinks.avro-sink.port = 44445

# Properties of sink logger-sink
agent_http.sinks.logger-sink.channel = ch2
agent_http.sinks.logger-sink.type = logger
```

第二级的 `Flume`配置如下，配置文件名为`flume-second.properties`：

```properties
# The configuration file needs to define the sources, the channels and the sinks.
# Sources, channels and sinks are defined per agent

agent_http.sources = collection
agent_http.channels = ch1 ch2
agent_http.sinks = kafka-sink logger-sink

agent_http.sources.collection.selector.type=replicating

# Properties of source
agent_http.sources.collection.channels = ch1 ch2
agent_http.sources.collection.type = avro
agent_http.sources.collection.bind = 192.168.100.100
agent_http.sources.collection.port = 44445
agent_http.sources.collection.handler = com.pengtuo.handler.ExampleHandler

# Properties of channel ch1
agent_http.channels.ch1.type = memory
agent_http.channels.ch1.capacity  = 1000
agent_http.channels.ch1.transactionCapacity  = 100

# Properties of channel ch2
agent_http.channels.ch2.type = memory
agent_http.channels.ch2.capacity  = 1000
agent_http.channels.ch2.transactionCapacity  = 100

# Properties of sink kafka
agent_http.sinks.kafka-sink.channel = ch1
agent_http.sinks.kafka-sink.type = org.apache.flume.sink.kafka.KafkaSink
agent_http.sinks.kafka-sink.kafka.topic = pengtuo-topic
agent_http.sinks.kafka-sink.kafka.bootstrap.servers = 192.168.100.100:9092
agent_http.sinks.kafka-sink.kafka.flumeBatchSize = 20
agent_http.sinks.kafka-sink.kafka.producer.acks = 1
agent_http.sinks.kafka-sink.kafka.producer.linger.ms = 1
agent_http.sinks.kafka-sink.kafka.producer.compression.type = snappy

# Properties of sink logger
agent_http.sinks.logger-sink.channel = ch2
agent_http.sinks.logger-sink.type = logger
```

> **注意**：
>
> 1. `Flume` 中多个 `channel` 与 多个 `sink` 之间是一个空格想隔开，**不是逗号！不是逗号！不是逗号！**
> 2. 配置里的 `agent_http.sources.collection.handler` 属性不是必要配置，如果没有自定义开发`handler`，可以删除这项。

### 1.2 启动与测试
启动脚本位于 `bin` 目录下，启动第一级命令为：

```shell
bin/flume-ng agent -c conf -f conf/flume-first.properties -n agent_http -Dflume.root.logger=INFO,console
```

参数解释：

- `-c or --conf`：表示用配置文件启动；
- `-f or --conf-file`：表示需要使用的配置文件，支持相对路径与绝对路径；
- `-n or --name`：表示`agent`的名字，例如本次为`agent_http`；

启动第二级命令同上，注意修改配置文件路径。

启动后可以执行测试数据发送命令进行测试：

```shell
curl --data "[{"headers":{"date":"2018-09-27", "value": 1}, "body":"hello"}]" 192.168.100.100:44444
```

> 这里贴的是 `Flume` 默认的 `Json` 数据格式，可自行修改。

这样在向端口号 `192.168.100.100:44444` 发送数据时，便可以通过控制台的输出来看数据流动到哪一层级，并且最后流向 `Kafka` 接收。

## 二、Kafka
现在进行 `Kafka` 配置来接收数据并且进行分发。

### 2.1 配置
下载 `Kafka` ，下载地址：[http://kafka.apache.org/downloads](http://kafka.apache.org/downloads)

`Kafka` 的版本与相应的 `Scala` 版本匹配，所以之前需要配置好`Scala`，本文档的版本选择为`scala-2.11.12`与`kafka_2.11-2.0.0`。

解压后，环境变量可看个人心情配置，贴出配置：

```properties
# set Kafka
export KAFKA_HOME=/Users/pengtuo/app/kafka_2.11-2.0.0
export PATH=$KAFKA_HOME/bin:$PATH
```

然后修改下述配置文件，`kafka`的配置文件都位于`kafka_2.11-2.0.0/config`下。

#### - server.properties 文件

此配置文件用来配置 `kafka` 服务器，介绍几个最基础的配置

1. `broker.id` ：声明当前 `kafka` 服务器在集群中的唯一ID，需配置为整数，并且集群中的每一个kafka服务器的id都应是唯一的，我们这里采用默认配置即可

2. `listeners` ：声明此 `kafka` 服务器需要监听的端口号，如果是在本机上跑虚拟机运行可以不用配置本项，默认会使用localhost的地址，如果是在远程服务器上运行则必须配置，例如：
```properties
listeners=PLAINTEXT://192.168.100.100:9092
```

3. `zookeeper.connect`：声明 kafka所连接的zookeeper的地址 ，需配置为zookeeper的地址，由于本次使用的是kafka高版本中自带zookeeper，使用默认配置即可
```properties
zookeeper.connect=192.168.100.100:2181
```

### 2.2 启动与测试

#### - 单 Kafka 测试
1. 启动 `zookeeper`
```shell
bin/zookeeper-server-start.sh config/zookeeper.properties
```

2. 启动 `kafka`
```shell
bin/kafka-server-start.sh config/server.properties
```

3. 测试 `kafka` 是否启动成功

开启两个终端窗口作为**生产者**和**消费者**。

在生产者窗口输入：
```shell
bin/kafka-console-producer.sh --broker-list 192.168.100.100:9092 --topic test1
```

然后在消费者窗口输入：
```shell
bin/kafka-console-consumer.sh --bootstrap-server 192.168.100.100:9092 --topic test1
```

则在生产者窗口输入『消息』在消费者窗口会有显示，如图：

![](http://omqlv3air.bkt.clouddn.com/blog/2018-10-15-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-10-15%20%E4%B8%8B%E5%8D%884.50.46.png)

![](http://omqlv3air.bkt.clouddn.com/blog/2018-10-15-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-10-15%20%E4%B8%8B%E5%8D%884.50.58.png)

此时表示 `kafka` 启动成功可以运行。

#### - Flume 与 Kafka 联调测试
最后一级的 `Flume` 配置 `kafka-sink` 时有指定

```properties
agent_http.sinks.kafka-sink.kafka.topic = pengtuo-topic
```

所以我们需要修改消费者的订阅 `topic`，执行以下启动命令即可：

```shell
bin/kafka-console-consumer.sh --bootstrap-server 192.168.100.100:9092 --topic pengtuo-topic
```

现在向第一级的 `Flume` 发送数据，则应该会在 `consumer` 输出相应的数据，此时表示以及连通。

### 2.3 错误处理
#### A) Flume 报错：`Error while fetching metadata with correlation id : {LEADER_NOT_AVAILABLE}`
**解决**：需要在 `server.properties` 文件中添加

```properties
host.name=192.168.100.100

listeners=PLAINTEXT://192.168.100.100:9092

advertised.listeners=PLAINTEXT://192.168.100.100:9092
```

#### B) Zookeeper 报错： `Error:KeeperErrorCode = NodeExists for /config/topics`
**解决**：可以不解决，不影响 `Kafka` 的使用，`zookeeper` 节点管理可以输入：

```shell
./zookeeper-shell.sh localhost:2181
```

然后  `ls / `来查看当前 `zookeeper` 中有哪些节点，删除节点则为 `rmr /{name}`。

## 三、 Spark
现在配置 `Spark` 端。

### 3.1 Maven 创建项目
> 推荐使用工具 `intellij idea`，利用 `maven` 新建 `spark-kafka-test` 项目。

修改 `pom.xml` 文件导入依赖，以下亲测可用：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.pengtuo</groupId>
    <artifactId>spark-kafka-test</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.11</artifactId>
            <version>2.1.1</version>
        </dependency>
    </dependencies>

</project>
```

然后在 `src/main/java `里创建自己的包，包里创建 `SparkKafkaTest.java` 文件，输入以下代码，解释见注释：
```java
package com.pengtuo;

import java.io.Serializable;
import java.util.*;

import org.apache.log4j.Logger;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.spark.streaming.kafka010.OffsetRange;

/**
 * class SparkKafkaTest
 *
 * @author pengtuo
 * @date 2018/10/15
 */
public class SparkKafkaTest implements Serializable {
    private static final Logger LOGGER = Logger.getLogger(SparkKafkaTest.class);

    public void analyze() {

        // Set kafka conf
        // 要订阅的 topics
        Collection<String> topicsSet = Arrays.asList("test1", "gehua-topic");

        Map<String, Object> kafkaParams = new HashMap<>();
        kafkaParams.put("metadata.broker.list", "0") ;
        // 此处 IP 需要与 Kafka IP 相同
        kafkaParams.put("bootstrap.servers", "192.168.100.101:9092");	
        kafkaParams.put("group.id", "group1");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("auto.offset.reset", "latest");
        kafkaParams.put("enable.auto.commit", false);

        // 设置 topic
        OffsetRange[] offsetRanges = {
                // topic, partition, inclusive starting offset, exclusive ending offset
                OffsetRange.create("gehua-topic", 0, 0, 100),
                OffsetRange.create("gehua-topic", 1, 0, 100)
        };

        // Set spark context
        final SparkConf sparkConf = new SparkConf().setMaster("local[2]").setAppName("Streaming kafka test");
        final JavaSparkContext sparkContext = new JavaSparkContext(sparkConf);
        sparkContext.setLogLevel("WARN");

        // 利用 KafkaUtils 获取 Kafka 中的数据转为 RDD
        JavaRDD<ConsumerRecord<String, String>> rdd = KafkaUtils.createRDD(
                sparkContext,
                kafkaParams,
                offsetRanges,
                LocationStrategies.PreferConsistent()
        );

        // 循环输出 RDD 里的内容
        rdd.foreach(new VoidFunction<ConsumerRecord<String, String>>() {
            @Override
            public void call(ConsumerRecord<String, String> stringStringConsumerRecord) throws Exception {
                System.out.println(stringStringConsumerRecord.toString());
            }
        });
    }

    public static void main(String[] args) {
        SparkKafkaTest test = new SparkKafkaTest();
        test.analyze();
    }
}

```
启动该程序便会输出一开始从 `Flume` 端输入的数据，此时整体框架的 `demo` 版便搭建好了。

## 参考
[1] [Spark Streaming + Kafka Integration Guide (Kafka broker version 0.10.0 or higher)](http://spark.apache.org/docs/2.2.0/streaming-kafka-0-10-integration.html)