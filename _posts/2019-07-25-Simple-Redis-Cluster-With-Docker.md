--- 
layout: post
title: "用docker 在本地搭一个简易redis 集群"
description: ""
category: "2019-07"
tags: [缓存,Redis,集群]
---

根据Redis cluster 的高可用规范，一个集群最少有6个节点，其中三个master，三个slave。那如果我们在本地测试的时候，有没有可能只有三个master 就可以呢？

> Note that the minimal cluster that works as expected requires to contain at least three master nodes. For your first tests it is strongly suggested to start a six nodes cluster with three masters and three slaves.

其实是可以的，只需要三个master 节点，要做的需要就是：

* 用docker 启动三个redis ；
* 在其中一个redis 中使用cluster meet 命令；
* 在每一个redis 上分配slots


### docker 启动Redis 

在用docker 启动Redis 时需要先做一下两件事情：

* 创建一个docker 网络：`docker network create redis_cluster`
* 创建Redis 集群节点配置文件：redis.conf，

如下是文件内容。

	port 6379
	cluster-enabled yes
	cluster-config-file nodes.conf
	cluster-node-timeout 5000
	appendonly yes

之后我们就可以启动三个Redis 实例了。

	docker run --name redis-1 --net redis_cluster -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf  -p 6379:6379 -d redis:5.0.3-alpine3.9 redis-server /usr/local/etc/redis/redis.conf && \
	docker run --name redis-2 --net redis_cluster -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf  -p 6380:6379 -d redis:5.0.3-alpine3.9 redis-server /usr/local/etc/redis/redis.conf && \
	docker run --name redis-3 --net redis_cluster -v $PWD/cluster-config.conf:/usr/local/etc/redis/redis.conf  -p 6381:6379 -d redis:5.0.3-alpine3.9 redis-server /usr/local/etc/redis/redis.conf

在启动Redis 实例时，通过--net 指定了网络，redis-server 命令指定了Redis 的配置文件。这样这三个节点都是以集群的模式启动。`1:M 22 Jul 2019 02:54:52.082 * Running mode=cluster, port=6379.`

### 加入集群

上面一步只是把集群的三个节点启动了，还没有创建集群，现在需要把三个节点加入到一个集群中。

因为三个节点都加入了`redis_cluster` 这个，需要先找到对应的ip。

	docker inspect redis-1
	docker inspect redis-2

我这里的ip 分别是：172.21.0.2，172.21.0.3

在redis-3节点中分别输入如下命令:

	CLUSTER MEET 172.21.0.3 6379
	CLUSTER MEET 172.21.0.2 6379

这时候三个节点都加入到redis-3 的集群中去了，但是集群还没有启动起来。因为slots 还没有分配。

	2bd6c5250d0113eab7aca7b288e84aaf5ced49da 172.21.0.2:6379@16379 master - 0 1564021852248 0 connected
	065d5b869cd765a171580f035ed23a113c63ec60 172.21.0.4:6379@16379 myself,master - 0 1564021851000 1 connected
	2bb4a925953cee563b80373ba87f31176aba7f6a 172.21.0.3:6379@16379 master - 0 1564021851742 2 connected

	cluster_state:fail
	cluster_slots_assigned:0
	cluster_slots_ok:0
	cluster_slots_pfail:0
	cluster_slots_fail:0
	cluster_known_nodes:3
	cluster_size:0
	cluster_current_epoch:2
	cluster_my_epoch:1
	cluster_stats_messages_ping_sent:6
	cluster_stats_messages_pong_sent:9
	cluster_stats_messages_meet_sent:2
	cluster_stats_messages_sent:17
	cluster_stats_messages_ping_received:9
	cluster_stats_messages_pong_received:8
	cluster_stats_messages_received:17


### 分配槽点

我们要把0-5000 到槽点分配到redis-1 节点，5001-10000 分配到redis-2 槽点，10001-16383 分配到redis-3节点。分别进入三个多节点容器中执行如下命令就可以了。

	# redis-1
	redis-cli -h 127.0.0.1 cluster addslots $(seq 0 5000)

	# redis-2
	redis-cli -h 127.0.0.1 cluster addslots $(seq 5001 10000)

	# redis-3 
	redis-cli -h 127.0.0.1 cluster addslots $(seq 10001 16383)


槽点分配完后，集群就会自动上线了。执行一下 `CLUSTER INFO` 看看：

	cluster_state:ok
	cluster_slots_assigned:16384
	cluster_slots_ok:16384
	cluster_slots_pfail:0
	cluster_slots_fail:0
	cluster_known_nodes:3
	cluster_size:3
	cluster_current_epoch:2
	cluster_my_epoch:1
	cluster_stats_messages_ping_sent:612
	cluster_stats_messages_pong_sent:615
	cluster_stats_messages_meet_sent:2
	cluster_stats_messages_sent:1229
	cluster_stats_messages_ping_received:615
	cluster_stats_messages_pong_received:614
	cluster_stats_messages_received:1229



### Reference

[cluster-tutorial](https://redis.io/topics/cluster-tutorial)
