## Presto

- ```bash
  # worker
  coordinator=false
  #node-scheduler.include-coordinator=true
  http-server.http.port=30085
  query.max-memory=5GB
  query.max-memory-per-node=1GB
  query.max-total-memory-per-node=2GB
  #discovery-server.enabled=true
  discovery.uri=http://10.180.210.24:30085
  jmx.rmiregistry.port=30086
  jmx.rmiserver.port=30087
  
  # coordinator
  coordinator=true
  node-scheduler.include-coordinator=true
  http-server.http.port=30085
  query.max-memory=5GB
  query.max-memory-per-node=1GB
  query.max-total-memory-per-node=2GB
  discovery-server.enabled=true
  discovery.uri=http://10.180.210.24:30085
  jmx.rmiregistry.port=30086
  jmx.rmiserver.port=30087
  
  # node
  node.environment=production
  node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
  node.data-dir=/home/presto/data
  
  # need java version > java 8u151+ !!，install 8u271
  
  # inject policy: airlyft
  
  
  # ./presto1.jar --server 10.180.210.24:2181 --catalog hive --schema default
  
  ```
## official
  # The following is a minimal configuration for the coordinator:

  coordinator=true
  node-scheduler.include-coordinator=false
  http-server.http.port=8080
  query.max-memory=50GB
  query.max-memory-per-node=1GB
  query.max-total-memory-per-node=2GB
  discovery-server.enabled=true
  discovery.uri=http://example.net:8080

  # And this is a minimal configuration for the workers:

  coordinator=false
  http-server.http.port=8080
  query.max-memory=50GB
  query.max-memory-per-node=1GB
  query.max-total-memory-per-node=2GB
  discovery.uri=http://example.net:8080

  ```
  
- bash
  # hdfs dfs -setrep -R 3 /  ## 设置文件复制级别。
  http-server.http.port=8001
Query 20201109_073157_00003_iw9ex failed: line 1:1: Catalog 'hive' does not exist
  
  ```

- ``` bash
  #
  presto-mysql
  <1>在 /etc/catalog/目录下创建mysql.properties, 在catalog下创建目录。
  
  connector.name=mysql
  
  connection-url=jdbc:mysql://localhost:3306
  
  connection-user=root
  
  connection-password=root
  
  ## mysql 数据表连接成功。观察可得：用户所使用的query, 可以瞬时发现和查找。
  
  # version： 0.242-BFC43C2
  # enter mysql cli
  ./presto-cli --server 10.180.210.24:30085 --catalog mysql  --schema test
  
  use test;
  show tables;
  select * from test1;
  select * from test1 limit 1;
  
  
  ```

- ![1604970412117](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1604970412117.png)

- ``` bash
  # enter hive
  ./presto-cli --server 10.180.210.24:30085 --catalog hive-hadoop2  --schema default
  
  ./presto-cli --server 10.180.210.24:30085 --catalog hive  --schema default
  
  ```

./presto-cli --server 10.180.210.24:30085 --catalog jmx --schema current ## load  data to local

  ```
  
- [Presto 介绍](https://blog.csdn.net/ycy0706/article/details/109520032)

二：
先解释下各参数的含义：

--server 是presto服务地址；
--catalog 是默认使用哪个数据源，后面也可以切换，如果想连接mysql数据源，使用mysql数据源名称即可；
--user 是用户名；
--source 是代表查询来源，source设置格式为key=value形式（英文分号分割）； 例如个人从command line查询应设置为pf=adhoc;client=cli。


- ``` bash
  CREATE TABLE IF NOT EXISTS tasks ( task_id INT  PRIMARY KEY, title VARCHAR(255) NOT NULL,start_date DATE,due_date DATE,status TINYINT NOT NULL,priority TINYINT NOT NULL,description TEXT) ;
  
  CREATE TABLE  `runoob_tbl`(
     `runoob_id` INT  ,
     `runoob_title` VARCHAR(100) NOT NULL,
     `runoob_author` VARCHAR(40) NOT NULL,
     `submission_date` DATE,
     PRIMARY KEY ( `runoob_id` )
  ) ENGINE=InnoDB;
  
  CREATE  TABLE `dept`(  
    `dept_no` int,   
    `addr` string,   
    `tel` string)
  partitioned by(date string ) 
  ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ; 
  
  #
  ./bin/hive --service metastore &
  
  # hive can access and also be read;
  
  ```

- [Hive 启动的相关命令](https://blog.csdn.net/u011120550/article/details/80599270)

- [Presto cli 介绍](https://github.com/yxydde/Presto-CLI)

- [Hbase 常见命令](https://www.cnblogs.com/nexiyi/p/hbase_shell.html)

- ``` bash
  # enter hbase mode
  hbase shell
  ```

- [Hbase connector issues(https://github.com/prestodb/presto/issues/3992)

- ```
  issues:
  
  java.lang.IllegalArgumentException: No factory for connector phoenix
  	at com.google.common.base.Preconditions.checkArgument(Preconditions.java:216)
  	at com.facebook.presto.connector.ConnectorManager.createConnection(ConnectorManager.java:208)
  	at com.facebook.presto.metadata.StaticCatalogStore.loadCatalog(StaticCatalogStore.java:123)
  	at com.facebook.presto.metadata.StaticCatalogStore.loadCatalog(StaticCatalogStore.java:98)
  	at com.facebook.presto.metadata.StaticCatalogStore.loadCatalogs(StaticCatalogStore.java:80)
  	at com.facebook.presto.metadata.StaticCatalogStore.loadCatalogs(StaticCatalogStore.java:68)
  	at com.facebook.presto.server.PrestoServer.run(PrestoServer.java:135)
  	at com.facebook.presto.server.PrestoServer.main(PrestoServer.java:77)
  
  
  
  ```

- [GC1- 介绍](https://tech.meituan.com/2016/09/23/g1.html)

- [dd 命令介绍](http://c.biancheng.net/view/1146.html)

- ``` bash
  # jmx connetion test passed;
  ```

- [presto 和 prestodb 的对比](https://zhuanlan.zhihu.com/p/87621360)

- [ES 的部署](https://www.cnblogs.com/aubin/p/8012840.html)

- ![1605058331233](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605058331233.png)

- ``` bash
  curl -XPUT 'localhost:9200/twitter?pretty' -H 'Content-Type:application/json' -d'{
  "settings":{
  "index":{
  "number_of_shards":3,
  "number_of_replicas":2
  }
  }
  '
  
  curl -XPUT '10.180.210.24:9200/twitter1?pretty' -H 'Content-Type:application/json' -d'{"settings":{"index" : {"number_of_shards" :3, "number_of_replicas" : 2}}'
  # support es also; -- but not able to fettch data;
  
  
  ```

- 345 - version - for prestosql --- download complete;

- ``` bash
  # redis connected;
  CREATE TABLE 'mysql_to_redis' (
    'id' int(11) NOT NULL AUTO_INCREMENT,
    PRIMARY KEY ('id')
  ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
  
  # enter to redis server mode
  redis-cli -c -h 10.180.210.24 -p 7001
  hgetall # get all
  dbsize # get the size
  config get databases # get db info
  
  ```

- [Hbase + phoenix use ](https://blog.csdn.net/sinadrew/article/details/79916723)

- ``` bash
  # start the phoenix
  ./sqlline.py 10.180.210.24,10.180.210.228,10.180.210.36:2181
  ```

- [zoo keeper status check](https://www.jianshu.com/p/3602921b5ee9)

- ``` bash
  # check zk status
  ./zkServer.sh status 
  ./zkCli.sh -server 10.180.210.24:2181  # check the port
  ```

- [Hbase failed to connect with zk](https://blog.csdn.net/zjh_746140129/article/details/83417879)

- [Hadoop, Hbase, Hive, Zookeepr default port](https://www.cnblogs.com/hankedang/p/5649414.html)

- [presto  - inner mechaniseum & misc from meituan](https://tech.meituan.com/2014/06/16/presto.html)

- [Presto db & presto sql compare](https://zhuanlan.zhihu.com/p/262236892)

- [Presto sanity checks](https://blog.csdn.net/qq_41504585/article/details/108424931)

- [Phoenix + Hbase](https://segmentfault.com/a/1190000011297889)

- ``` xml
  <property>
      <name>hbase.table.sanity.checks</name>
      <value>false</value>
  </property>
  <property>
      <name>hbase.regionserver.wal.codec</name>		       <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
  </property>
  <property>
      <name>phoenix.schema.isNamespaceMappingEnabled</name>
      <value>true</value>
  </property>
  <property>
      <name>phoenix.schema.mapSystemTablesToNamespace</name>
      <value>true</value>
  </property>
  <property>
      <name>phoenix.coprocessor.maxServerCacheTimeToLiveMs</name>
      <value>300000000</value>
  </property>
  <property>
      <name>hbase.regionserver.executor.openregion.threads</name>
      <value>100</value>
  </property>
  ```

- ``` bash
  # 
  connector.name=redis
  redis.nodes=localhost:6379
  
  redis.hide-internal-columns=false
  redis.table-description-dir=/home/redis
  redis.table-names=user_info
  redis.default-schema=user_profile
  redis.key-delimiter=:
  redis.key-prefix-schema-table=true
  redis.hide-internal-columns=false
  
  # get db info
  info keyspace
  get config databases
  
  # list all redis patterns
  keys user* 
  
  # get user info
  select * from redis.user_profile.user_info;
  
  
  ```

  

  

- ``` bash
  # 
  # prestosql-- ansi sql is diff from hive sql; -- HQL
  
  # change to double division;
  SELECT CAST(5 AS DOUBLE) / 2
  
  #
  {
     "tableName": "cytest",
  
     "schemaName": "redis",
  
     "key": {
         "dataFormat": "raw",
  
         "fields": [
  
              {
                  "name":"redis_key",
  
                  "dataFormat":"_default",
  
                  "type":"VARCHAR",
  
                  "hidden":"false"
  
             }
  
         ]
  
     },
  
     "value": {
         "dataFormat": "json",
  
         "fields": [
  
             {
                  "name":"row_number",
  
                  "mapping":"rowNumber",
  
                  "type":"BIGINT"
  
             },
  
             {
                  "name":"customer_key",
  
                  "mapping":"customerKey",
  
                  "type":"BIGINT"
  
             },
  
             {
                  "name":"name",
  
                  "mapping":"name",
  
                  "type":"VARCHAR"
  
             }
  
        ]
  
      }
  
  }
  
  #
  
  set key2"{\"rowNumber\":3,\"customerKey\":3,\"name\":\"Customer#000000003\"}"https://prestosql.io/docs/current/develop/connectors.html
  
  #  hive -- chmod -R 777 /warehouse ;
  
  ```

- [Connector customd ways](https://prestosql.io/docs/current/develop/connectors.html)

- ``` bash
  # kafka connector, check the topics
  kafka-topics.sh --list --zookeeper localhost:2181 
  
  # get the topic total infor: 
  ./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list manager93.bigdata:6667 --topic topic-test1 --time -1
  
  # consumer cli
  ./kafka-console-consumer.sh --bootstrap-server 10.180.210.93:6667 --topic topic-test1
  
  # producer cli
  ./kafka-console-producer.sh --broker-list manager93.bigdata:6667,master94.bigdata:6667 -topic topic-test1 
  # note
  pay attention to the topic replica, should large than the replicas;
  
  # exam topic details
  ./kafka-topics.sh --list --zookeeper localhost:2181
  
  #
  
  ```

- [kafka connector tutorial](https://prestosql.io/docs/current/connector/kafka-tutorial.html)

- ``` bash
  # download the generated data
  $ curl -o kafka-tpch https://repo1.maven.org/maven2/de/softwareforge/kafka_tpch_0811/1.0/kafka_tpch_0811-1.0.sh
  $ chmod 755 kafka-tpch
  
  # kafka config 
  
  # test select query;
  SELECT _message FROM customer LIMIT 5;
  
  # check the sum of account
  SELECT sum(cast(json_extract_scalar(_message, '$.accountBalance') AS double)) FROM customer LIMIT 10;
  ```

- ![1605583719858](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605583719858.png)

- ![1605583766897](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605583766897.png) the presto check cli
- ![1605583853481](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605583853481.png)

- ![1605592930530](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605592930530.png) show all the separated columns;

- ``` bash
  # set up the live twitter messages
  $ curl -o twistr https://repo1.maven.org/maven2/de/softwareforge/twistr_kafka_0811/1.2/twistr_kafka_0811-1.2.sh
  $ chmod 755 twistr
  
  
  ```

- 1. Functions and Operators;

     ```
     
     ```

     ##



## Functions and Operations

- ```mysql
  SELECT CAST(null AS boolean) AND true; -- null
  
  SELECT CAST(null AS boolean) AND false; -- false
  
  SELECT CAST(null AS boolean) AND CAST(null AS boolean); -- null
  ```

- ``` mysql
  # presto lang syntax;
  Create a new schema web in the current catalog:
  
  CREATE SCHEMA web
  Create a new schema sales in the hive catalog:
  
  CREATE SCHEMA hive.sales
  Create the schema traffic if it does not already exist:
  
  CREATE SCHEMA IF NOT EXISTS traffic
  Create a new schema web and set the owner to user alice:
  
  CREATE SCHEMA web AUTHORIZATION alice
  Create a new schema web, set the LOCATION property to /hive/data/web and set the owner to user alice:
  
  CREATE SCHEMA web AUTHORIZATION alice WITH ( LOCATION = '/hive/data/web' )
  ```

- ![1605664884634](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1605664884634.png) Alluxio's component

- [Presto with alluxio](https://xie.infoq.cn/article/2dfcdffd66b91664c68c64701)

- ``` bash
  # show all the system nodes;
  SELECT * FROM system.runtime.nodes;
  ```

- ``` bash
  # windows add ssh port forwarding: check & get
  Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Client*'
  Add-WindowsCapability -Online -Name OpenSSH.Client*
  
  # windows ssh tunnel forwarding to remote port way, 90 is the remote host, port 3389 is remote, 8888 is the local machine. also recommend use powershell to do the task;
  ssh -L 8888:192.168.1.90:3389 root@192.168.1.90
  ```

- ``` bash
  # suite 3 versions:
  Linux-aarch64
  Linux-ppc64le
  Linux-x86_64
  
  #
  ERROR: Presto requires Java 11+ (found 1.8.0_271)
  ```

- [JVM 堆内存分配](https://www.cnblogs.com/wang1001/p/9556444.html)

- ``` bash
  -- # hive operation
  ALTER TABLE table_name RENAME TO new_table_name
  
  # change table name;
  ALTER TABLE tgm_test RENAME TO tgm_test1;
  # rename column, type -- support int and varchar
  ALTER TABLE tgm_test1 RENAME column  name to name1;
  # add column
  ALTER TABLE tgm_test1 add column  namess1  varchar;
  # drop column
  ALTER TABLE tgm_test1 drop  column  namess;
  
  # from hive to mysql - left join
  select * from hive.hivetest.tgm_test1 as tg left join mysql.test11.t1 as t2 on tg.id = t2.id where tg.id =1;
  
  
  # Hive support file format
  - ORC
  
  - Parquet
  
  - Avro
  
  - RCText (RCFile using ColumnarSerDe)
  
  - RCBinary (RCFile using LazyBinaryColumnarSerDe)
  
  - SequenceFile
  
  - JSON (using org.apache.hive.hcatalog.data.JsonSerDe)
  
  - CSV (using org.apache.hadoop.hive.serde2.OpenCSVSerde)
  
  TextFile
  
  # drop schema
  DROP SCHEMA hive.web;
  
  ## mysql insert
  insert into t1 (id) values (1);
  insert t1 (1);
  
  ## cross origin data source join
  use hivetest;
  show tables;
  # left join
  select * from hive.hivetest.tgm_test1 as tg left join mysql.test11.t1 as t2 on tg.id = t2.id where tg.id =1;
  
  ## mysql need to be the cluster;
  
  #
  通过 Presto 执行流程的架构，可以看出 Presto 在查询上也存在一些不足：
  1.没有容错能力，当一个 query 分发到多个 Worker 去执行时，当有一个 Worker 因为各种原因查询失败，Master 感知到之后，整个 query 也会失败。
  2.内存限制，由于 Presto 是纯内存计算，所以当内存不够时，Presto 并不会将结果 dump 到磁盘上，所以查询也就失败了。
  3.并行查询，因为所有的 task 都是并行执行，如果其中一台 Worker 因为各种原因查询很慢，那么整个 query 就会变得很慢。
  4.并发限制，因为全内存操作+内存限制，能同时处理的数据量有限，因而导致并发能力不足。
  ```

- ``` bash
  # presto 查询流程：
  查询流程如下：
  
  Client 使用 HTTP 协议发送一个 query 请求。
  通过 Discovery Server 发现可用的 Server。
  Coordinator 构建查询计划（通过 Anltr3 解析为 AST（抽象语法树），然后通过 Connector 获取原始数据的 Metadata 信息，生成分发计划和执行计划）。
  Coordinator 向 Worker 发送任务。
  Worker 通过 Connector 插件读取数据。
  Worker 在内存里执行任务（Worker 是纯内存型计算引擎）。
  Worker 将数据返回给 Coordinator，汇总之后再响应客户端。
  ```



## Presto 源码阅读

``` bash

```

[- antr generator plugin usage](https://www.jianshu.com/p/21f2afca65e8)

   The way to generate the .g4 java files, also need to add import packages.

- idea can choose specific module to build

- [maven Enforcer  plugin](https://stackoverflow.com/questions/45337558/failed-to-execute-goal-org-apache-maven-pluginsmaven-enforcer-plugin)

- mvn package -Dmaven.test.skip=true -Dcheckstyle.skip=true  , skip the test and checkstyle part;

- ``` bash
  # unzip
  jar -xvf test.jar
  
  # zip
   jar -cvf xxx.jar a.class b.class
   # all zip
   jar -cvf xx.jar *
  ```

- [Ambari_flink - integration](http://10.180.210.148:8080/zhangyao/ambari_flink)

- ``` bash
  # Ambari _ presto
  
  # define the installation location
  chmod -R 777 /path/to/presto-server-345.tar.gz
  mkdir -p /var/www/html/InsightHD/hdp/presto/
  cp /path/to/presto.tar.gz /var/www/html/InsightHD/hdp/presto/
  cp /path/to/jdk11 /usr/jdk64/
  
  # ambari - install
  cp -r /presto/install/files /var/lib/ambari-server/resources/stacks/HDP/3.0/services/PRESTO 
  ambari-server restart
  
  # how to distributed the worker nodes
  
  # how to use j2 templates
  
  # how to monitor the service -- application
  
  # how to activate the jdk 11 env
  modify launcher , add java path to launcher
  
  # 
  10.180.210.93/ljk3991!
  
  
  
  ```

- ``` bash
  # list all the shell env
  chsh -l  
  lha命令是从lharc演变而来的压缩程序，文件经它压缩后，会另外产生具有.lzh扩展名的压缩文件。
  ```

  * [Ambari basics](https://ambari.apache.org/1.2.1/installing-hadoop-using-ambari/content/ambari-chap1-1.html)

*  [JVM OPT 介绍](https://www.jianshu.com/p/bb6b91f2fb29)

* ``` absh
  five steps:
  
  - install
  - start
  - stop
  - status
  - config
  - main
  
  ## the deploy management scripts:
  /usr/lib/ambari-server/lib/resource_management/libraries/functions
  locale -a # list all locale
  #
  uuidgen # generate uuid
  sed -i 's/node.id=mmmffffffff-ffff-ffff-ffff-ffffffffffff/node.id=$(uuidgen)/g' node.properties 
  
  #
  
  1.清理前内存使用情况 
  free -m
  
  2.开始清理  
  echo 1 > /proc/sys/vm/drop_caches
  echo 3 > /proc/sys/vm/drop_caches
  3.清理后内存使用情况 
  free -m
  
  # su -hdfs
  
  hadoop 之hdfs数据块修复方法：
  1、手动修复
  hdfs fsck / #检查集群的健康状态
  hdfs debug recoverLease -path 文件位置 -retries 重试次数 #修复指定的hdfs数据块。也就是关闭打开的文件。
  
  检查坏块：
  hadoop fsck /user -files -blocks -locations
  
  检查是否有数据块正在写入：
  hadoop fsck /user openforwrite
  
  ```

* [Hibench 使用](https://www.cnblogs.com/hellowcf/p/6912746.html)

* ``` bash
  ## implement the zip format;
  
  // 开启map端输出压缩
  configuration.setBoolean("mapreduce.map.output.compress", true);
  // 设置map端输出压缩方式
  configuration.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
  ```

* [Zip compress](https://blog.csdn.net/qq_40947493/article/details/104295757)

* ``` bash
  # yarn.app.mapreduce.am.resource.mb
  
  echo -e "line1\r\nline2" >> .bashrc ## add new line carriage - return;
  
  ## call link - for templates usage. - The standard usage ways:
   from metainfo.xml -> config/presto-conf.xml -> params.py -> config.properties.j2 -> realfolder/conf/presto-conf.properties. Then server call with config files.
   
   
   # add the resourcemanagement path;
   import sys
   sys.path.insert(0, "/usr/lib/ambari-server/lib/")
   from resource_management import *
   # 
   
   # https://www.cnblogs.com/zhbzz2007/p/6158125.html - context lib 上下文管理器。
  
  ```

- [Presto 集成Kerboros 环境下的Hive](https://cloud.tencent.com/developer/article/1158362)
- core  -> resources ->system.py -- includes the Link etc functions.
- [Python C3 MRO 集成](https://www.cnblogs.com/whatisfantasy/p/6046991.html)， use inspect MRO wisely.
- [Presto 介绍](https://zhuanlan.zhihu.com/p/101366898)
- 