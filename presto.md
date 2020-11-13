## Presto

- ```bash
  # worker
  coordinator=false
  node-scheduler.include-coordinator=true
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

- ``` bash
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
  
./presto-cli --server 10.180.210.24:30085 --catalog jmx --schema current ## load  data to local
  
  ```
  
- [Presto 介绍](https://blog.csdn.net/ycy0706/article/details/109520032)

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
- 