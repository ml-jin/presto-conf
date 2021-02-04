# 华为hetu 手工部署文档

[TOC]



## 手工河图部署

## 依赖版本

Hetu （openLooKeng): 1.1.0

Java JDK: 11.0.9

## 安装河图

服务器上传hetu tar 包。 

``` bash
hetu-server-1.1.0.tar.gz
```



## 配置hetu(别名openLooKeng)

根目录下创建etc文件夹，内部创建以下配置文件：

- Node Properties: 特定于每个节点的环境配置
- JVM Config: Java 虚拟机的命令行选项
- Config Properties: openLooKeng 服务器的配置
- Catalog Properties: [连接器] 的配置（数据源）

### 节点属性

节点属性文件`etc/node.properties`包含特定于每个节点的配置。*节点* 是计算机上的 openLooKeng 的单个已安装实例。此文件通常由首次安装 openLooKeng 时的部署系统创建。以下是最小 `etc/node. properties`: （其中worker节点一次调整其node.id, 例如最后两位改为f1 ~ f9 等）

```properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.data-dir=/var/openlookeng/data
```

- 上述属性如下所述：

  - "node.environment"：环境的名称。群集中的所有 openLooKeng 节点必须具有相同的环境名称。
  - "node.id"：此安装 openLooKeng 的唯一标识符。这必须对于每个节点是唯一的。此标识符应在 openLooKeng 的重新启动或升级期间保持一致。如果在一台计算机上运行多个 openLooKeng 安装（即同一台计算机上的多个节点），则每个安装必须具有唯一标识符。
  - "node.data-dir"：数据目录的位置（文件系统路径）。openlooKeng 将在这里存储日志和其他数据。

### JVM 配置

JVM 配置文件"etc/jvm.config"包含用于启动 Java 虚拟机的命令行选项列表。文件的格式是选项列表，每行一个。这些选项不由 shell 解释，因此不应引用包含空格或其他特殊字符的选项。

下面提供了创建"etc/jvm.config"的一个很好的建议配置：

```properties
-server
-Xmx16G
-XX:-UseBiasedLocking
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+UseGCOverheadLimit
-XX:+HeapDumpOnOutOfMemoryError
-XX:ReservedCodeCacheSize=512M
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
```

由于"OutOfMemoryError"通常会使JVM处于不一致状态，因此我们编写一个堆转储（用于调试），并在发生这种情况时强制终止进程。

### Config 属性

配置属性文件等/配置属性包含 openLooKeng 服务器的配置。每个 openLooKeng 服务器都可以同时充当协调器和辅助人员，但将一台计算机用于仅执行协调工作可在较大的群集上提供最佳性能。

以下是协调器的最小配置：

```properties
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=50GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://example.net:8080
```


以下是工作人员的最小配置：

或者，如果您要设置一台用于测试的机器，该计算机将同时作为协调器和辅助程序运行，请使用此配置：

```properties
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8080
query.max-memory=5GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://example.net:8080
```

- 这些属性需要一些解释：

  - `coordinator`：允许此 openLooKeng 实例充当协调器（接受来自客户端的查询并管理查询执行）。
  - `node-scheduler.include-coordinator`：允许对协调器进行调度工作。对于较大的群集，协调器上的处理工作可能会影响查询性能，因为计算机的资源不能用于调度、管理和监视查询执行等关键任务。
  - `http-server.http.port`：指定 HTTP 服务器的端口。openLooKeng 使用 HTTP 进行所有内部和外部通信。
  - `query.max-memory`：查询可能使用的最大分布式内存量。
  - `query.max-memory-per-node`：查询在任何一台计算机上可能使用的最大用户内存量。
  - `query.max-total-memory-per-node`：查询在任何一台计算机上可以使用的最大用户和系统内存量，其中系统内存是读取器、编写器和网络缓冲区等执行期间使用的内存。
  - `discovery-server.enabled`：openLooKeng 使用发现服务查找群集中的所有节点。每个 openLooKeng 实例都将在启动时向发现服务注册。为了简化部署并避免运行附加服务，openLooKeng 协调员可以运行发现服务的嵌入式版本。它将 HTTP 服务器与 openLooKeng 共享，因此使用相同的端口。
  - `discovery.uri`：发现服务器的 URI。由于我们已经在 openLooKeng 协调器中启用了发现的嵌入式版本，因此这应该是 openLooKeng 协调器的 URI。更换"example.net:8080"以匹配 openLooKeng 协调器的主机和端口。此 URI 不得以斜杠结束。

可以设置以下属性：

- "jmx.rmiregistry.port"：指定 JMX RMI 注册表的端口。JMX 客户端应连接到此端口。
- "jmx.rmiserver.port"：指定 JMX RMI 服务器的端口。openLooKeng 导出许多用于通过 JMX 进行监视的指标。

另请参阅 [资源组]。

### Log 等级配置

可选的日志级别文件"etc/log.properties"允许为命名记录器层次结构设置最小日志级别。每个记录器都有一个名称，该名称通常是使用记录器的类的完全限定名称。记录器具有基于名称中的点（如 Java 包）的层次结构。例如，请考虑以下日志级别文件：

```properties
io.prestosql=INFO
```

这将为"io.prestosql.server"和"io.prestosql".plugin"设置最低级别。默认最低级别为"INFO"。有四个级别："调试"，"信息"，"警告"和"错误"。

### Catalog 属性

openLooKeng 通过 *连接器* 访问数据，该连接器安装在目录中。连接器提供目录内的所有架构和表。例如，Hive 连接器将每个 Hive 数据库映射到架构，因此，如果 Hive 连接器装载为"Hive"目录，并且 Hive 在数据库"web"中包含一个表"单击"，则该表将在 openLooKeng 中作为"Hive.web.点击"进行访问。

目录通过在"etc/目录"目录中创建目录属性文件进行注册。例如，使用以下内容创建"etc/目录/jmx.属性"，以将"jmx"连接器装载为"jmx"目录：

```properties
connector.name=jmx
```

有关配置连接器的更多信息，请参阅 [连接器]

## 运行 Hetu -  openLooKeng

在bin/launcher 中修改代码，添加两行：

```bash
vim bin/launcher
# + 2 第38行 =>
PATH=/opt/presto/jdk-11.0.9/bin/:$PATH
export PATH
```

注：原生代码需移除plugin/ 下的carbondata 目录，否则会抛java异常导致无法启动。

安装目录包含"bin/启动器"中的启动器脚本。openLooKeng 可以通过运行以下操作作为守护进程启动：

```shell
bin/launcher start
```

或者，它可以在前台运行，日志和其他输出被写入 stdout/stderr（如果使用像damontools这样的监督系统，应捕获两个流）：

```shell
bin/launcher run
```

- 使用"-help"运行启动器以查看支持的命令和命令行选项。特别是，"-详细"选项对于调试安装非常有用。

  启动后，您可以在"var/log"中找到日志文件：

  - `launcher.log`：此日志由启动器创建，并连接到服务器的粗壮和粗线流。它将包含一些在初始化服务器日志记录时发生的日志消息以及 JVM 产生的任何错误或诊断。
  - ``server.log``：这是 openLooKeng 使用的主日志文件。如果服务器在初始化过程中发生故障，它通常会包含相关信息。它会自动旋转和压缩。
  - ```http-request.log```：这是包含服务器接收的每个 HTTP 请求的 HTTP 请求日志。它会自动轮询和压缩。

若最后可以输入http://coor_ip:30088, 可以访问页面，则说名部署成功。