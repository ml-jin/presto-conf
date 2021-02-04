# Hetu 开发者指南

[TOC]

## 开始

本文档为用户提供了在本地主机上快速部署和启动 openLooKeng 服务的指导。有关安装要求和部署模式的详细信息，请参阅安装。

还提供演示环境。[现在就试试吧](https://tryme.openlookeng.io/)

准备环境:
选择 Linux 作为操作系统。
内存> 4 GB
机器连接到互联网
端口号 8090 未占用。

** 一键式部署 **

运行以下命令以启动安装和部署：
``` bash
wget -O - https://download.openlookeng.io/install.sh|bash
```
此命令用于从 openLooKeng 官方网站下载安装脚本并启动脚本。在脚本运行期间，最新的安装包和从属组件会自动从 openLooKeng 官方网站下载。下载完成后，部署将自动启动。无需额外操作。

如果显示以下日志，则已成功部署 openLooKeng

![1612251118639](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1612251118639.png)

成功部署后，您可以学习以下信息，以便更好地使用 openLooKeng 服务。

默认情况下，以下内置数据源配置为 openLooKeng 一键部署：

- [tpcds](https://openlookeng.io/docs/docs/connector/tpcds.html)
- [tpch](https://openlookeng.io/docs/docs/connector/tpch.html)
- [memory](https://openlookeng.io/docs/docs/connector/memory.html)

openLooKeng 的安装路径是 /opt/openlookeng。您可以在此处找到 openLooKeng 的配置文件。有关配置文件和配置项的信息，请参阅此处。

将创建新的用户 openlkadmin 以执行与 openLooKeng 相关的操作，包括启动/停止 openLooKeng 服务以及扩展/减少群集规模。

一键式部署还提供常见管理命令的脚本。你可以在这里找到/选择/打开看/宾。

openLooKeng 运行日志存储在 /home/openlkadmin/.openlkadmin/日志中。

一键部署还提供用于连接到 openLooKeng 服务的 CLI。

## 总揽

## 安装

## 安全

## 管理

## 查询优化

## 连接器

## 方法和运算符