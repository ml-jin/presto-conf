# ZooKeeper 验证

[TOC]

验证zookeeper是否成功启动

bin/zkServer.sh status（一个leader，两个follower）

或者在Zookeeper安装的任何一个节点执行客户端连接命令：

bin/zkCli.sh -server 192.168.1.1:2181