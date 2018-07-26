---
title: EMQ安装与配置
date: 2018-07-26 09:53:34
tags:
---

# MQTT简介

1. 基于二进制消息的发布/订阅模式的消息协议
2. 适合需要低功耗和网络带宽有限的IoT场景：
   - 遥感数据、汽车、智能家居、智慧城市、医疗医护

# MQTT对比

| server      | 性能                    | 收费      | 集群     |
| ----------- | --------------------- | ------- | ------ |
| mosquitto   | 测试例子7000左右            | 开源免费    | 不支持    |
| HiveMQ      | 支持到千万级  亚毫秒级的延迟  高吞吐量 | 根据设备数收费 | 支持     |
| VerneMQ     | 支持百万级                 | 开源免费    | 无主集群技术 |
| emqttd（EMQ） | 支持单节点100万连接  毫秒级低时延   | 开源免费    | 分布式集群  |

emqttd和VerneMQ：

性能上相差不大，都是基于Erlang/OTP分布式编程

emqttd当前有已知的稳定的应用（ZAKER新闻客户端）

# EMQ安装及配置

##下载路径

[http://](http://emqtt.com/downloads)[emqtt.com/downloads](http://emqtt.com/downloads)

## 启动方法

```
unzip emqttd-ubuntu14.04-v2.2.0.zip
cd emqttds
sudo ./bin/emqttd start
```

##配置

EMQ2.0消息服务器通过etc/目录下配置文件进行设置，

主要配置文件包括:

1. etc/emq.conf            --->   EMQ 消息服务器配置文件
2. etc/acl.conf               --->   EMQ 默认ACL规则配置文件
3. etc/plugins/*.conf   --->   EMQ 各类插件配置文件

日志参数配置

![log](EMQ安装与配置\log.png)

