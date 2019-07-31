---
layout:     post
title:      "Flume 初识与搭建"
subtitle:   "Meet and build Flume"
date:       2018-10-16
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-flume-01.jpg"
categories: [Big Data]
tags: Flume
---
本文概览：
* TOC
{:toc}

## 1 什么是 Flume

`Flume` 是一种分布式，可靠且可用的服务，用于有效地收集，聚合和移动大量日志数据。它具有基于流数据流的简单灵活的架构，还具有可靠的可靠性机制和许多故障转移和恢复机制，具有强大的容错能力。它使用简单的可扩展数据模型，允许在线分析应用程序。

`Apache Flume` 的使用不仅限于日志数据聚合。 由于数据源是可定制的，因此 `Flume` 可用于传输大量事件数据，包括但不限于网络流量数据，社交媒体生成的数据，电子邮件消息以及几乎任何可能的数据源。

## 2 Flume 架构

### 2.1 数据流模型

在 `Flume` 中流动的数据是被封装成了 `event` 对象，又可称之为事件，一个 `Flume` 事件被定义为承载字节数据以及可选字符串属性集的数据流单元。 `Flume` 代理是一个（`JVM`）进程，它承载事件从外部源流向下一个目标。

`Flume` 的数据流通路径如图：

![](https://live.staticflickr.com/65535/48420208276_937a33a653_o.png)

其大体模块分为三个部分，分别是 `Source`(源端)、`Channel`(通道) 以及 `Sink`(接收器)。细分则还有 `Interceptor`(数据拦截器，位于 `Source` 与 `Channel` 之间)，`Channel Selector`(通道选择器，当有多个通道时，可通过配置确定数据的流向)，`Sink Processor`(处理器，位于 `Channel` 与 `Sink` 之间) 以及 `Serialization`(序列化，处于和 `Sink` 同样的位置)。

`Flume` 的 `Source` 端接收从其他数据源（例如网络服务器）传入的数据，然后封装成 `event`。当 `Flume` 源接收事件时，它将其转发到一个或多个 `Channel`。通道是一个被动存储器，可以保持事件直到它被 `Flume Sink` 成功接收。接收器从通道中移除事件并将其放入外部存储库（如 `HDFS`，通过 `Flume HDFS` 接收器））或将其转发到流中下一个 `Flume` 代理（下一跳）的源端。

> `Flume` 代理程序中的源和接收器与通道中暂存的事件异步运行，允许用户构建多层代理，并且支持负载均衡和故障转移。

`Flume` 中的数据流单位是 `event`，包含 `header` 和 `body` 两个部分，`header` 是 `map` 结构，可在其中以 `key-value` 的形式存放字符串属性集，`body` 则是字节流，用于存放数据的内容。如下图：

![](https://live.staticflickr.com/65535/48420210026_385c0109d7_o.png)

#### 2.1.1 可靠性

事件在每个代理的通道中进行，然后将事件传递到流中的下一个代理或终端存储库（如 `HDFS`）。只有将事件存储在下一个代理的通道或终端存储库中后，才会从通道中删除这些事件。所以确保了在一个 `Flume` 节点中数据传递的可靠性。

`Flume` 使用事务方法来保证事件的可靠传递。源和接收器分别在事务中封装事件的存储与检索，这可确保事件集在流中从一个点到另一个点可靠地传递。在多跳流的情况下，来自前一跳的接收器和来自下一跳的源都运行其事务以确保数据安全地存储在下一跳的信道中。

#### 2.1.2 可恢复性

事件在通道中进行，由通道管理从故障中恢复。 `Flume` 支持一个由本地文件系统支持的持久化文件通道。其也支持内存通道，但它只是将事件存储在内存中的队列中，运行速度更快，但是当代理进程宕了时仍然留在内存通道中的事件将无法恢复。

### 2.2 数据收集部件 Source

`Source`是数据的收集端，负责将数据捕获后进行特殊的格式化，将数据封装到事件（`event`） 里，然后将事件推入`Channel`中，`Flume`还支持自定义`Source`。

内置的支持的`source`类型有：

| Name                       | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| Avro Source                | 支持Avro协议（实际上是Avro RPC）                             |
| Thrift Source              | 支持Thrift协议                                               |
| Exec Source                | 基于Unix的command在标准输出上生产数据                        |
| JMS Source                 | 从JMS系统中读取数据                                          |
| Spooling Directory Source  | 监控指定目录内数据变更                                       |
| Twitter 1% firehose Source | 通过API持续下载Twitter数据，试验性质                         |
| Netcat Source              | 监控某个端口，将流经端口的每一个文本行数据作为Event输入      |
| Sequence Generator Souvrce | 序列生成器数据源，生产序列数据                               |
| Syslog Sources             | 读取syslog数据，产生Event，支持UDP和TCP两种协议              |
| HTTP Source                | 基于HTTP POST或GET方式的数据源，支持JSON、BLOB表示形式(实际上支持任何形式，因为handle可以自定义) |
| Legacy Sources             | 兼容老的Flume OG中Source（0.9.x版本）                        |

### 2.3 数据流通部件 Channel

`Channel` 是一种短暂的存储容器,负责数据的存储持久化，可以持久化到 `jdbc, file, memory`，将从`source`处接收到的`event`格式的数据缓存起来,直到它们被`sinks`消费掉，可以把`channel`看成是一个队列。

数据只有存储在下一个存储位置（可能是最终的存储位置，如`HDFS`；也可能是下一个`Flume`节点的`channel`），数据才会从当前的`channel`中删除。这个过程是通过事务来控制的，这样就保证了数据的可靠性。

`Flume`内置的`Channel`如下：

| Name                       | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| Memory Channel             | 内存通道                                                     |
| JDBC Channel               | 存储在持久化存储中，当前Flume Channel内置支持Derby           |
| File Channel               | 存储在磁盘文件中                                             |
| Spillable Memory Channel   | 存储在内存中和磁盘上，当内存队列满了，会持久化到磁盘文件（当前试验性的，不建议生产环境使用） |
| Pseudo Transaction Channel | 测试用途                                                     |

### 2.4 数据转发部件 Sink

`sink` 负责数据的转发，将数据存储到集中存储器比如`Hbase`和`HDFS`,它从`channel`消费数据(`events`)并将其传递给目标地，如下一个 `agent` 或者是文件系统。

`Flume`内置的`sink`如下：

| Name                | Description                                         |
| ------------------- | --------------------------------------------------- |
| HDFS Sink           | 数据写入HDFS                                        |
| Logger Sink         | 数据写入日志文件                                    |
| Avro Sink           | 数据被转换成Avro Event，然后发送到配置的RPC端口上   |
| Thrift Sink         | 数据被转换成Thrift Event，然后发送到配置的RPC端口上 |
| IRC Sink            | 数据在IRC上进行回放                                 |
| File Roll Sink      | 存储数据到本地文件系统                              |
| Null Sink           | 丢弃到所有数据                                      |
| HBase Sink          | 数据写入HBase数据库                                 |
| Morphline Solr Sink | 数据发送到Solr搜索服务器（集群）                    |
| ElasticSearch Sink  | 数据发送到Elastic Search搜索服务器（集群）          |
| Kite Dataset Sink   | 写数据到Kite Dataset，试验性质的                    |

### 2.5 其他部件

#### 2.5.1 拦截器 Interceptor

拦截器的位置在`Source`和`Channel`之间，当我们为`Source`指定拦截器后，我们在拦截器中会得到`event`，根据需求我们可以对`event`进行保留还是抛弃，或者进行处理，抛弃的数据不会进入`Channel`中。

`Flume` 中目前提供了以下拦截器：

1. Timestamp Interceptor：时间戳拦截器，将当前时间戳（毫秒）加入到 `events header` 中，`key`名字为：timestamp，值为当前时间戳
2. Host Interceptor：主机名拦截器。将运行Flume agent的主机名或者IP地址加入到 `events header` 中，key名字为：`host`（也可自定义）
3. Static Interceptor：静态拦截器，用于在 `events header` 中加入一组静态的`key`和`value`
4. UUID Interceptor：UUID拦截器，用于在每个`events header` 中生成一个UUID字符串，生成的`UUID`可以在`sink`中读取并使用
5. Search and Replace Interceptor：搜索和替换拦截器，该拦截器用于将events中的正则匹配到的内容做相应的替换
6. Regex Filtering Interceptor：正则拦截器，通过正则来清洗或包含匹配的events
7. Regex Extractor Interceptor：通过正则表达式来在`header`中添加指定的 `key`，`value` 则为正则匹配的部分

用法示例：

```properties
a1.sources.r1.interceptors=i1 i2  
a1.sources.r1.interceptors.i1.type=regex_filter  
a1.sources.r1.interceptors.i1.regex=\\{.*\\}  
a1.sources.r1.interceptors.i2.type=timestamp
```

可以同时指定多个拦截器。

#### 2.5.2 通道选择器 Channel Selector

通道选择器的主要功能是对事件流进行复制和分流；

`Flume`内置了两种类型的通道选择器：

1. 复制（`Replicating Channel Selector`），使用该选择器，我们可以同时让同一事件传递到多个`channel`中，最后流入多个`sink`；
2. 分流（`Multiplexing Channel Selector`），使用该选择器，我们可以让特定的事件流入到特定的channel中，如不同项目产生的日志事件，交由不同的sink处理；

#### 2.5.3 转发处理器 Sink Processor

`Sink processors` 能够提供在组内所有`Sink`之间实现负载均衡的能力，而且在失败的情况下能够进行故障转移从一个`Sink`到另一个`Sink`。

官网配置的例子：

```properties
a1.sinkgroups=g1
a1.sinkgroups.g1.sinks=k1 k2
a1.sinkgroups.g1.processor.type=load_balance
```

`Flume`提供了三种：

1. default sink processor：接收单一的`Sink`，不强制用户为`Sink`创建Processor。

2. failover sink processor：FailoverSink Processor会通过配置维护了一个优先级列表。保证每一个有效的事件都会被处理，故障转移的工作原理是将连续失败sink分配到一个池中，在那里被分配一个冷冻期，在这个冷冻期里，这个sink不会做任何事。一旦sink成功发送一个event，sink将被还原到live 池中。

   官网配置：

   | Property Name                     | Default | Description                                                  |
   | --------------------------------- | ------- | ------------------------------------------------------------ |
   | **sinks**                         | -       | Space-separated list of sinks that are participating in the group |
   | **processor.type**                | default | The component type name, needs to be failover                |
   | **processor.priority.<sinkName>** | -       | <sinkName> must be one of the sink instances associated with the current sink group |
   | processor.maxpenalty              | 30000   | (in millis)                                                  |

   > 加粗黑体的属性是必须要设置的

   案例：

   ```properties
   a1.sinkgroups=g1
   a1.sinkgroups.g1.sinks=k1 k2
   a1.sinkgroups.g1.processor.type=failover
   a1.sinkgroups.g1.processor.priority.k1=5
   a1.sinkgroups.g1.processor.priority.k2=10
   a1.sinkgroups.g1.processor.maxpenalty=10000
   ```

3. Load balancing Sink Processor：负载均衡片处理器提供在多个Sink之间负载平衡的能力。实现支持通过`round_robin`（轮询）或者`random`（随机）参数来实现负载分发，默认情况下使用`round_robin`，但可以通过配置覆盖这个默认值。还可以通过集成`AbstractSinkSelector`类来实现用户自己的选择机制。

   官网配置：

   | Property Name                 | Default       | Description                                                  |
   | ----------------------------- | ------------- | ------------------------------------------------------------ |
   | **processor.sinks**           | -             | Space-separated list of sinks that are participating in the group |
   | **processor.type**            | `default`     | The component type name, needs to be `load_balance`          |
   | processor.backoff             | false         | Should failed sinks be backed off exponentially.             |
   | processor.selector            | `round_robin` | Selection mechanism. Must be either `round_robin`, `random` or FQCN of custom class that inherits from `AbstractSinkSelector` |
   | processor.selector.maxTimeOut | 30000         | Used by backoff selectors to limit exponential backoff (in milliseconds) |

   > 加粗黑体的属性是必须要设置的

   官网案例：

   ```properties
   a1.sinkgroups = g1
   a1.sinkgroups.g1.sinks = k1 k2
   a1.sinkgroups.g1.processor.type = load_balance
   a1.sinkgroups.g1.processor.backoff = true
   a1.sinkgroups.g1.processor.selector = random
   ```

#### 2.5.4 序列化部件 Serialization

`Sink`负责将事件输出到外部，那么以何种形式输出（直接文本形式还是其它形式），需要包含哪些东西（`body`还是`header`还是其它内容...），就是由事件序列化来完成的；

`Flume`内置的事件序列化如下：

1. Body Text Serializer：看名字就知道，直接将事件的`body`作为文本形式输出，事件`header`将被忽略

   | Property Name | Default | Description                                                  |
   | ------------- | ------- | ------------------------------------------------------------ |
   | appendNewline | true    | Whether a newline will be appended to each event at write time. The default of true assumes that events do not contain newlines, for legacy reasons. |

   案例：

   ```properties
   a1.sinks = k1
   a1.sinks.k1.type = file_roll
   a1.sinks.k1.channel = c1
   a1.sinks.k1.sink.directory = /var/log/flume
   a1.sinks.k1.sink.serializer = text
   a1.sinks.k1.sink.serializer.appendNewline = false
   ```

2. Avro Event Serializer：Avro序列化，包含事件全部信息

`Flume`支持自定义事件序列化，需要实现`EventSerializer`接口。

## 3 Flume 部署搭建

### 3.1 安装

首先从官网下载 `Flume`，下载地址为 [http://www.apache.org/dyn/closer.lua/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz](http://www.apache.org/dyn/closer.lua/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz)

然后执行解压命令：

```sh
tar -zxvf apache-flume-1.8.0-bin.tar.gz -C ~/app
```

> 看过前面的[Hadoop 伪分布式环境搭建](http://pengtuo.tech/%E5%A4%A7%E6%95%B0%E6%8D%AE%E7%A0%94%E5%8F%91/2018/09/04/hadoop-pseudo-distributed/)的同学应该会知道，个人建议下载的压缩包都放置在 `~/software/` 中，解压后放置在 `~/app/` 中。

然后配置环境变量，在相应的文件（`MacOS`是`.zshrc`文件，`Linux`是`.bashrc`或`.profile`文件）中添加以下代码：

```properties
# set Flume
export FLUME_HOME=/Users/pengtuo/app/apache-flume-1.8.0-bin
export PATH=$FLUME_HOME/bin:$PATH
```

其中的路径改成自己的 `Flume` 文件路径，修改退出后记得执行 `source .zshrc`。

> 是否配置环境变量可以依个人心情，配置环境变量的好处是可以在任何目录下启动 `Flume`。

### 3.2 配置

配置文件的路径在`$FLUME_HOME$/conf`目录下，初始安装时，目录里只有`flume-conf.properties.template`文件，首先将其改名为`.properties`结尾，然后修改其中的值。

我这里列出一个从网络端口接收消息，经过 `Flume` 后传入 `HDFS` 内储存的配置案例，文件名为`flume-conf-hdfs.properties`：

```properties
# The configuration file needs to define the sources, the channels and the sinks.
# Sources, channels and sinks are defined per agent

agent_hdfs.sources = collection
agent_hdfs.channels = mem-channel
agent_hdfs.sinks = hdfs-sink

# define the flow
agent_hdfs.sources.collection.channels = mem-channel
agent_hdfs.sinks.hdfs-sink.channel = mem-channel

# Properties of source
agent_hdfs.sources.collection.type = netcat
agent_hdfs.sources.collection.bind = localhost
agent_hdfs.sources.collection.port = 44444

# Properties of channel
agent_hdfs.channels.mem-channel.type = memory
agent_hdfs.channels.mem-channel.capacity = 1000
agent_hdfs.channels.mem-channel.transactionCapacity = 100

# Properties of sink
agent_hdfs.sinks.hdfs-sink.type = hdfs
agent_hdfs.sinks.hdfs-sink.hdfs.path = hdfs://pengtuo-Mac:8020/flume
agent_hdfs.sinks.hdfs-sink.hdfs.minBlockReplicas = 1
agent_hdfs.sinks.hdfs-sink.hdfs.useLocalTimeStamp = true
agent_hdfs.sinks.hdfs-sink.hdfs.round = false
agent_hdfs.sinks.hdfs-sink.hdfs.filePrefix = test_log_%H
agent_hdfs.sinks.hdfs-sink.hdfs.rollInterval = 3600
agent_hdfs.sinks.hdfs-sink.hdfs.rollCount = 0
agent_hdfs.sinks.hdfs-sink.hdfs.rollSize = 134217728
agent_hdfs.sinks.hdfs-sink.hdfs.batchSize = 1000
agent_hdfs.sinks.hdfs-sink.hdfs.fileType = DataStream
agent_hdfs.sinks.hdfs-sink.hdfs.writeFormat = Text

```

其中配置里的`agent_hdfs.sinks.hdfs-sink.hdfs.path`需要改为自己的`hdfs`文件路径。

### 3.3 运行

单级启动，只需要执行

```shell
flume-ng agent -c conf -f /Users/pengtuo/app/apache-flume-1.8.0-bin/conf/flume-conf-hdfs.properties -n agent_http -Dflume.root.logger=INFO,console
```

即可启动。

## 4 参考

1 [http://flume.apache.org/FlumeUserGuide.html](http://flume.apache.org/FlumeUserGuide.html)

2  [https://www.cnblogs.com/chenpi/p/7240584.html#_label1](https://www.cnblogs.com/chenpi/p/7240584.html#_label1)