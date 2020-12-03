---
title: Elasticsearch从入门到放弃(二) -- 零基础环境搭建
date: 2018-01-28 9:33:35
tags:
- Elasticsearch
- ES
- 数据库
- 搜索引擎
---
本文主要讲述 Linux 虚拟机环境下 Elasticsearch 5.5 版本的安装。安装 Elasticsearch 之前，请确保你的机器 Java 8 环境已经搭建好，保证环境变量 JAVA_HOME 设置正确。如你还没有安装好 JAVA 8 环境，请参考[linux系统中JAVA环境搭建](/2017/11/04/java_1-1/)

#### 下载并解压安装包
``` bash
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.0.zip
$ unzip elasticsearch-5.5.0.zip
$ cd elasticsearch-5.5.0/ 
```
#### 修改配置文件
Elasticsearch 默认只允许本机访问，远程访问，需要修改 config/elasticsearch.yml,
改为运行所有人访问 0.0.0.0（生产环境，切不可这样改，可以指定特定 ip 访问 Elasticsearch）
``` bash
$ vim config/elasticsearch.yml

#network.host: 192.168.0.1
network.host: 0.0.0.0
```
#### 启动程序
直接运行 bin 目录下的 elasticsearch
``` bash
$ ./bin/elasticsearch
```
此过程中，可能会报出以下错误
``` bash
max virtual memory areas vm.maxmapcount [65530] is too low
```
此时，切换到管理员用户，运行下面的命令即可
``` bash
# sysctl -w vm.max_map_count=262144
```
重新运行 elasticsearch 即可，这时候访问 9200 端口，得到如下信息：
``` bash
$ curl http://127.0.0.1:9200
{
  "name" : "uA7Io-i", # node 名称
  "cluster_name" : "elasticsearch", # 集群名称 
  "cluster_uuid" : "f7y_hJefSw-kQFN-zt-3Cw", # 集群唯一 id
  "version" : {
    "number" : "5.5.0", # Elasticsearch 版本号
    "build_hash" : "260387d",
    "build_date" : "2017-06-30T23:16:05.735Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0" # 依赖的 Lucene 版本号
  },
  "tagline" : "You Know, for Search"
}
```
到这里说明你的 Elasticsearch 已经安装成功了，按下 Ctrl + C，Elasticsearch 就会停止运行。 想要后台运行 Elasticsearch，输入下面命令：
``` bash
$ ./bin/elasticsearch -d
```
此时想要停止 Elasticsearch ，想要先找到 Elasticsearch 进程，然后 kill 。
``` bash
$ ps -ef |grep elasticsearch

martin    3035  2200  0 Apr19 ?        00:04:29 /usr/bin/java -Xms2g -Xmx2g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -Djdk.io.permissionsUseCanonicalPath=true -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Dlog4j.skipJansi=true -XX:+HeapDumpOnOutOfMemoryError -Des.path.home=/home/marting/elasticsearch-5.5.0 -cp /home/martin/elasticsearch-5.5.0/lib/* org.elasticsearch.bootstrap.Elasticsearch

$ kill -2 3035
```