---
layout:     post
title:      "Hadoop入门（一）之 Hadoop 伪分布式环境搭建"
subtitle:   "Hadoop 挖坑第一篇"
date:       2018-09-04
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-hadoop-01.jpg"
categories: 大数据研发
tags:
    - Hadoop
    - HDFS
---

本文概览：
* TOC
{:toc}

以这篇文章开启大数据开发系列教程更新，本人也是努力学习中

## 1. 环境要求
首先 Java 版本不低于 Hadoop 相应版本要求，一般的，Hadoop 大版本号在 2.6 以前的支持 Java 6，Hadoop 大版本号 在 2.7 ~ 3.0 之间的支持 Java 7，Hadoop 版本在 3.0 之后的支持 Java 8

详细可见官网 [Hadoop Java Versions](https://wiki.apache.org/hadoop/HadoopJavaVersions)

本文所用的 Hadoop 版本为 `hadoop-2.6.0-cdh5.7.0`，这个版本很稳定，属于大多企业使用的 Hadoop 版本，Java 版本使用的是 `java version "1.7.0_80"`

在 Linux 中下载，执行以下命令：

- 下载 `Java 8`
```shell
wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz
```

- 下载 `Java 7`
```shell
wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz?AuthParam=1523684058_82b7921ee0def49bd2a0930187900e60
```

- 下载 `hadoop-2.6.0-cdh5.7.0`
```shell
wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0.tar.gz
```

> 本人的建议文件管理方式，将这些下载都存储到 `~/downloads/` 文件夹里，然后解压到 `~/app/` 文件里

## 2. 环境配置
#### 2.1 配置 Java
执行解压命令

```shell
tar -zxvf jdk-8u161-linux-x64.tar.gz -C ~/app/
```

创建`.profile` 文件，如果有就不用创建了，然后在`.profile` 文件里添加

```shell
export JAVA_HOME=/root/app/jdk1.7.0_80
export PATH=$JAVA_HOME/bin:$PATH
```

保存退出后，执行 `source .profile` 让配置生效

#### 2.2 配置 ssh 免密登录
HDFS 是由一个 `NameNode`，一个 `SecodaryNameNode`，以及 n 个 `DataNode` 组成，当有多台物理机时，`NameNode` 与 `DataNode` 是分布在不同的物理机上，部署则需要 `NameNode` 能够直接与 `DataNode` 进行通信，通信方式之一就是使用 SSH ([Secure Shell](https://baike.baidu.com/item/Secure%20Shell))，所以需要在之间设置免密登录

因为本次是 Hadoop 伪分布式搭建，本机同时充当 `NameNode` 与 `DataNode` 角色，所以只需要配置一个本机的 SSH 免密登录

执行：
```shell
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```
即可

#### 2.3 修改 hadoop 配置文件
解压 hadoop 压缩包：

```shell
tar -zxvf hadoop-2.6.0-cdh5.7.0.tar.gz -C ~/app/
```

在 `.profile` 文件里添加：
```shell
export HADOOP_HOME=/root/app/hadoop-2.6.0-cdh5.7.0
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$HADOOP_HOME/bin:$PATH
```

Hadoop 的配置文件都在 `hadoop_home/etc/hadoop/` 里，如果你的文件管理方式和我一样的话，则是在 `~/app/hadoop-2.6.0-cdh5.7.0/etc/hadoop/` 中

在 `hadoop-env.sh` 中添加：
```shell
export JAVA_HOME= 你的 java home 路径
```

在 `core-site.xml` 中添加：
```xml
<configuration>

<property>
    <name>fs.defaultFS</name>
    <value>hdfs:// {你的 IP 地址或 hostname} :8020</value>
</property>

<property>
    <name>hadoop.tmp.dir</name>
    <value>~/app/tmp</value>
</property>

</configuration>
```

在 `hdfs-site.xml` 中添加：
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 2.4 启动 hdfs
首先格式化文件系统

> 注意：仅第一次执行即可，不要重复执行

```shell
cd ~/app/hadoop-2.6.0-cdh5.7.0
bin/hdfs namenode -format
```

然后启动 `namenode` 和 `datanode`

```shell
sbin/start-dfs.sh
```

检验是否启动成功，执行 `jps`，如果显示：

```shell
3644 SecondaryNameNode
3341 NameNode
3450 DataNode
4141 Jps
```

则表示环境配置成功，如果少一个就表示启动没有成功，则需要检查相应的日志进行错误排查

## 3. 错误排查
`NameNode`、`SecondaryNameNode`以及`DataNode`的启动日志都在`~/app/hadoop-2.6.0-cdh5.7.0/logs/`中，查看对应的`.log`文件可获得启动信息以及错误日志

本人配置过程的遇到的错误有：

(1) 地址绑定错误：
```
Problem binding to [aliyun:8030] 
java.net.BindException: Cannot assign requested address; 
For more details see:  http://wiki.apache.org/hadoop/BindException

Caused by: java.net.BindException: Cannot assign requested address
```
解决方法：本人是在阿里云服务器上配置，在 `/etc/hosts` 文件中，主机名配置IP不能用公网IP，需要用内网IP

(2) 在启动 hadoop 时，有 `log4j` 的 `warning` 警告信息

解决方法：`vim etc/hadoop/log4j.properties`，然后添加 `log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR`

> 注意：此方法只是修改了 `log4j` 的活跃等级，并不是解决了 `warning` 的根源，属于治标不治本

(3) 执行 `jps` 后，只启动了 `NameNode` 与 `SecondaryNameNode`，没有启动 `DataNode`
查看日志得到错误日志：
```
2018-09-04 11:47:38,166 FATAL org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for Block pool <registering> (Datanode Uuid unassigned) service to aliyun/172.16.252.38:8020. Exiting.
java.io.IOException: All specified directories are failed to load.
	at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:478)
	at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:1394)
	at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:1355)
	at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:317)
	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:228)
	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:829)
	at java.lang.Thread.run(Thread.java:745)
```

这个是文件系统初试化时出了问题

解决方法：停止已启动的节点，停止命令为 `sbin/stop-dfs.sh`，删除 `~/app/tmp/dfs` 文件夹，然后重新到 `~/app/hadoop-2.6.0-cdh-5.7.0/` 执行 `bin/hdfs namenode -format`，然后启动 `sbin/start-dfs.sh`，此时就能够成功启动

> **强烈注意**：`bin/hdfs namenode -format` 是**格式化**文件系统命令，如果你是初次搭建，可以用此方法**暴力解决**，但是如果已经使用了 Hadoop 一段时间，HDFS 存在重要数据，则需要另找它法。

OK，后面将会讲解 Hadoop 的重要组成部分以及相关知识
