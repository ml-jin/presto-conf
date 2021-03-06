# Presto 操作指南

[TOC]

## 简介


Apache Presto是一个开源的分布式SQL引擎。 Presto起源于Facebook，用于数据分析需求，后来被开源。现在，Teradata加入了Presto社区并提供支持。

Apache Presto对于执行甚至PB级数据的查询非常有用。可扩展的体系结构和存储插件接口非常易于与其他文件系统进行交互。当今大多数最佳的工业公司都采用Presto，因为它具有交互式速度和低延迟性能。

本教程探讨了Presto的体系结构，配置和存储插件。它讨论了基本和高级查询，最后以实时示例结束。


## 概要
数据分析是分析原始数据以收集相关信息以做出更好决策的过程。它主要在许多组织中用于制定业务决策。好吧，大数据分析涉及大量数据，并且此过程非常复杂，因此公司使用不同的策略。

例如，Facebook是全球领先的数据驱动和最大的数据仓库公司之一。 Facebook仓库数据存储在Hadoop中以进行大规模计算。后来，当仓库数据增长到PB时，他们决定开发一种低延迟的新系统。在2012年，Facebook团队成员为交互式查询分析设计了“ Presto”，即使有PB级数据也可以快速运行。
- 什么时Presto？
  Apache Presto是一个分布式并行查询执行引擎，针对低延迟和交互式查询分析进行了优化。 Presto可以轻松地运行查询，并且可以在不停机的情况下进行扩展，甚至从千兆字节到PB级。
  一个Presto查询可以处理来自多个来源的数据，例如HDFS，MySQL，Cassandra，Hive和更多数据源。 Presto是用Java内置的，易于与其他数据基础架构组件集成。 Presto功能强大，Airbnb，DropBox，Groupon和Netflix等领先公司都在采用它。
- Presto − 特性
  Presto包含以下功能-
      1.简单和可扩展的体系结构。
      2.可插拔连接器-Presto支持可插拔连接器，以提供用于查询的元数据和数据。
      3.流水线执行-避免不必要的I / O延迟开销。
      4.用户定义的功能-分析师可以创建自定义的用户定义的功能以轻松迁移。
      5.向量化的柱状处理。
- Presto - 优势
  这是Apache Presto提供的好处的列表-
    1.专门的SQL操作
    2.易于安装和调试
    3.简单的存储抽象
    4.以低延迟快速扩展PB级数据
- Presto - 应用
  Presto支持当今大多数最佳的工业应用。让我们看一些著名的应用程序。
    1.Facebook-Facebook为数据分析需求而构建了Presto。 Presto可轻松扩展大规模数据。
    2.Teradata-Teradata在大数据分析和数据仓库中提供端到端解决方案。 Teradata对Presto的贡献使更多公司更容易满足所有分析需求。
    3.Airbnb-Presto是Airbnb数据基础架构不可或缺的一部分。好吧，每天都有数百名员工使用该技术运行查询。
- 为什么是presto？
  Presto支持标准的ANSI SQL，这使得数据分析师和开发人员非常容易。尽管它是用Java内置的，但是它避免了与内存分配和垃圾回收有关的Java代码的典型问题。 Presto具有Hadoop友好的连接器体系结构。它允许轻松地插入文件系统。
  Presto可在多个Hadoop发行版上运行。此外，Presto可以从Hadoop平台联系查询Cassandra，关系数据库或其他数据存储。这种跨平台的分析功能使Presto用户能够从千兆字节到PB的数据中提取最大的业务价值。
## 架构

## 安装

## 配置

## 管理

## SQL 操作

## SQL 功能

## Mysql 连接器

## JMX 连接器

## HIVE 连接器

## KAFKA 连接器

## JDBC 连接器

## 订制化应用

## Presto 常用资源



