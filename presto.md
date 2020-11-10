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

- 