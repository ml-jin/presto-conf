# Presto on Yarn 手工部署调试

[TOC]

## presto on yarn生产实践原

**presto on yarn方案缺点**

- hadoop集群的jdk版本过低无法满足prestoserver需求
- 配置文件不易维护，特别是如果涉及多个hadoop集群，这点在大公司很常见
- hadoop集群单独团队维护，自己没有权限创建目录之类的，这个也很常见
- 指定coordinator节点需要使用yarn label，否则每次重启都得找coordinator节点

针对上述问题，解决方案如下：hadoop集群jdk版本低的问题，可以尝试将高版本jdk打包到prestoserver中去，启动的时候用自带的jdk；配置文件也可以和prestoserver一起打包，不使用appConfig.conf这种方式配置；presto中的数据目录打算在程序slider启动时，在tmp目录创建，不再手动创建；presto  on yarn这种方式，我们只部署worker节点；coordinator单独找一台物理机部署
 **版本如下**

| hadoop version | presto version | slider version           | jdk version  |
| -------------- | -------------- | ------------------------ | ------------ |
| cdh5.15.2      | 0.223          | slider-0.80.0-incubating | jdk1.8.0_181 |

## 下载编译presto-yarn

```
git clone https://github.com/prestodb/presto-yarn.git
yum install -y maven
```

修改presto.verison和slider.version

```
vi pom.xml

<presto.version>0.223</presto.version>
<slider.version>0.80.0-incubating</slider.version>
```

[修改configure.py](http://xn--configure-z89nz78p.py)

```
#!/usr/bin/env python

from resource_management import *
import ast, os, shutil

def set_configuration(component=None):

    import params

    if (os.path.exists(format("{data_dir}"))):
        shutil.rmtree(format("{data_dir}"))

    if (os.path.lexists(format("{conf_dir}"))):
        os.remove(format("{conf_dir}"))

    _directory(params.pid_dir, params)
    _directory(params.log_dir, params)
 

    if params.addon_plugins:
        plugins_dict = ast.literal_eval(params.addon_plugins)
        for key, value in plugins_dict.iteritems():
            plugin_dir = os.path.join(params.presto_plugin_dir, key)
            if not os.path.exists(plugin_dir):
                os.makedirs(plugin_dir)
            for jar in value:
                shutil.copy2(os.path.join(params.source_plugin_dir, jar), plugin_dir)
            
def _directory(path, params):
    Directory(path,
              owner=params.presto_user,
              group=params.user_group,
              recursive=True
              )
```

修改presto_server.py

```
#!/usr/bin/env python

from resource_management import *
from configure import set_configuration
import os

class PrestoServer(Script):
    def __init__(self, component):
        self.component = component

    def install(self, env):
        self.install_packages(env)
        pass

    def configure(self):
        set_configuration(self.component)

    def start(self, env):
        import params

        env.set_params(params)

        self.configure()
        #修改此处
        # os.symlink(format('{conf_dir}'), os.path.join(format('{presto_root}'), 'etc'))
        os.symlink(os.path.join(format('{presto_root}'), 'etc'),format('{conf_dir}'))
        process_cmd = format("PATH={presto_root}/jdk1.8.0_181/bin:$PATH {presto_root}/bin/launcher run --node-config {conf_dir}/node.properties --jvm-config {conf_dir}/jvm.config --config {conf_dir}/config.properties >> {log_file} 2>&1")

        Execute(process_cmd,
                logoutput=True,
                wait_for_finish=False,
                pid_file=params.pid_file,
                poll_after=3
                )

    def stop(self, env):
        pass

    def status(self, env):
        import params

        env.set_params(params)
        check_process_status(params.pid_file)


if __name__ == "__main__":
    self.fail_with_error('Component name missing')
```

打包编译, 需要耐心等待编译。

```
mvn clean package -DskipTests
```

编译完后的包在`presto-yarn-package/target`目录下

```
/Users/xiao/IdeaProjects/presto-yarn/presto-yarn-package/target/presto-yarn-package-1.6-SNAPSHOT-0.223.zip
```

打完包后`presto-yarn-package-1.6-SNAPSHOT-0.223.zip`中的prestoserver用的是官网的安装包，不包含任何配置文件，所以需要解压后替换成包含配置文件的package，同时将jdk也放到prestoserver中去，然后重新打包，这个过程就不演示了,我的prestoserver package结构如下

```
[yarn@hadoop6 presto-server-0.223]$ ll
total 220
drwxr-xr-x  3 yarn yarn   4096 Jul 26 04:17 bin
drwxr-xr-x  3 yarn yarn   4096 Aug  7 11:45 etc
drwxr-xr-x  7 yarn yarn   4096 Aug  7 14:17 jdk1.8.0_181
drwxrwxr-x  2 yarn yarn  12288 Aug  7 09:41 lib
-rw-r--r--  1 yarn yarn 191539 Jul 26 03:58 NOTICE
drwxrwxr-x 30 yarn yarn   4096 Aug  7 09:41 plugin
-rw-r--r--  1 yarn yarn    126 Jul 26 03:58 README.txt
```

## 下载配置slider

地址`https://archive.apache.org/dist/incubator/slider/`,这里我们下载的版本是`0.80.0-incubating`,一定要选择和自己hadoop版本一致的版本，这里下载的是已经编译好的包

```
wget https://archive.apache.org/dist/incubator/slider/0.80.0-incubating/slider-assembly-0.80.0-incubating-all.zip
```

解压

```
unzip slider-assembly-0.80.0-incubating-all.zip
```

## 配置slider

```
vi slider-env.sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_181
export HADOOP_CONF_DIR=/etc/hadoop/conf
vi slider-client.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
      <name>slider.client.resource.origin</name>
      <value>conf/slider-client.xml</value>
      <description>This is just for diagnostics</description>
    </property>

    <property>
      <name>yarn.log-aggregation-enable</name>
      <value>true</value>
    </property>

    <property>
        <name>slider.security.protocol.acl</name>
      <value>*</value>
      <description>When security is enabled, set appropriate acl. Default value means allow everyone.</description>
    </property>

    <property>
      <name>yarn.application.classpath</name>
      <value>$HADOOP_CLIENT_CONF_DIR,$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/*,$HADOOP_COMMON_HOME/lib/*,$HADOOP_HDFS_HOME/*,$HADOOP_HDFS_HOME/lib/*,$HADOOP_YARN_HOME/*,$HADOOP_YARN_HOME/lib/*</value>
    </property>

    <property>
      <name>yarn.resourcemanager.address</name>
      <value>Hadoop:8032</value>
    </property>

    <property>
      <name>yarn.resourcemanager.scheduler.address</name>
      <value>hadoop:8030</value>
    </property>

    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://hadoop:8020</value>
    </property>

    <property>
      <name>hadoop.registry.zk.quorum</name>
      <value>hadoop1:2181,hadoop2:2181,hadoop3:2181,hadoop4:2181,hadoop5:2181</value>
    </property>

</configuration>
```

presto 包安装

```
[yarn@hadoop6 ~]$ slider-0.80.0-incubating/bin/slider package --install --name presto --package presto-yarn-package-1.6-SNAPSHOT-0.223.zip
2019-08-06 15:15:21,690 [main] INFO  client.SliderClient - Installing package file:/var/lib/hadoop-yarn/presto-yarn-package-1.6-SNAPSHOT-0.223.zip to hdfs://hadoop:8020/user/yarn/.slider/package/presto/presto-yarn-package-1.6-SNAPSHOT-0.223.zip (overwrite set to false)
2019-08-06 15:15:30,805 [main] INFO  client.SliderClient - Set application.def in your app config JSON to .slider/package/presto/presto-yarn-package-1.6-SNAPSHOT-0.223.zip
2019-08-06 15:15:30,807 [main] INFO  util.ExitUtil - Exiting with status 0
```

配置slider
 vi  slider-0.80.0-incubating/conf/appConfig.json

```bash
{
  "schema": "http://example.org/specification/v2.0.0",
  "metadata": {
  },
  "global": {
    "site.global.app_user": "yarn",
    "site.global.user_group": "hadoop",
    "site.global.data_dir": "/tmp/presto223/data",
    "site.global.config_dir": "/tmp/presto223/etc",
    "site.global.app_name": "presto-server-0.223",
    "site.global.app_pkg_plugin": "${AGENT_WORK_ROOT}/app/definition/package/plugins/",
    "site.global.singlenode": "false",
    "site.global.coordinator_host": "coodinatorhost 节点ip",
    "site.global.presto_server_port": "8092",
    "site.global.catalog":"",
    "application.def": ".slider/package/presto/presto-yarn-package-1.6-SNAPSHOT-0.223.zip"
  },
  "components": {
    "slider-appmaster": {
      "jvm.heapsize": "128M"
    }
  }
}
```

vi  slider-0.80.0-incubating/conf/resources.json

```bash
{
  "schema": "http://example.org/specification/v2.0.0",
  "metadata": {
  },
  "global": {
    "yarn.vcores": "1"
  },
  "components": {
    "slider-appmaster": {
    },
    "WORKER": {
      "yarn.role.priority": "2",
      "yarn.component.instances": "7",
      "yarn.component.placement.policy": "2",
      "yarn.memory": "12000"
    }
  }
}
```

**部署coordinator 节点**
 找台物理机单独部署coordinator，这部分略过

**启动worker**

```bash
#第一次执行忽略第一条命令，多次执行的时候需要删除hdfs上的缓存安装包
hdfs dfs -rmr -skipTrash hdfs://hadoop1:8020/user/yarn/.slider/cluster/prestoserver

./bin/slider create prestoserver --template conf/appConfig.json --resources conf/resources.json
```

查看slider日志，可以看到使用的是presto自带的jdk

```bash
2019-08-07 14:36:13,686 - Execute['PATH=/data2/yarn/nm/usercache/yarn/appcache/application_1565086656619_0056/container_1565086656619_0056_01_000008/app/install/presto-server-0.223/jdk1.8.0_181/bin:$PATH /data2/yarn/nm/usercache/yarn/appcache/application_1565086656619_0056/container_1565086656619_0056_01_000008/app/install/presto-server-0.223/bin/launcher run --node-config /tmp/presto223/etc/node.properties --jvm-config /tmp/presto223/etc/jvm.config --config /tmp/presto223/etc/config.properties >> /tmp/presto223/data/var/log/server.log 2>&1'] {'pid_file': '/tmp/presto223/data/var/run/slider_launcher.pid', 'wait_for_finish': False, 'logoutput': True, 'poll_after': 3}
```

控制台
 ![在这里插入图片描述](http://blog.gqylpy.com/media/ai/2019-09/e8102162-1a17-44aa-a0b9-2b18566810d5.png)

问题

1:python环境问题
 要确保python环境统一，这里使用系统自带的
 Python 2.6.6 (r266:84292, Aug 18 2016, 15:13:37)
 [GCC 4.4.7 20120313 (Red Hat 4.4.7-17)] on linux2
 Type “help”, “copyright”, “credits” or “license” for more information.
 如果有些节点一直部署不成功，检查yarn container日志，很有可能是python环境问题，例如错误：CERTIFICATE_VERIFY_FAILED

2.启动的server实例少于配置所指定
 原因有很多，在我配置过程中总结了下大概有如下几点：

- 端口被占用，主要是由于一个nodemanger启动多个presto的containner，通常我会让container的内存大于nodemanager最大内存一半以上，这样每个nodemanager就只能启动一个prestoserver
- yarn没有足够的资源分配给prestoserver，主要体现在nodemanager的可用内存不够启动prestoserver，也有可能是nodemanager cpu 全被占用，这些都是可以在控制台查看：http://hadoop2:8088/cluster/nodes

3.日志查看

通常情况下slider日志提供的信息非常有限，尽量查看yarn的日志，`yarn logs -applicationId appid`

- [Hetu](https://openlookeng.io/)

- [Hetu channel](https://openlookeng.slite.com/p/channel/EDMAZKydV2MsM5trxJPmLv#)

https://openlookeng.io/docs/docs/installation/deployment.html -- deploy 

- [MR - 介绍](http://10.7.11.229:8080/kms/multidoc/kms_multidoc_knowledge/kmsMultidocKnowledge.do?method=view&fdId=177433158bbf77bd16645ae43ff883dc)
- [MR - 架构](https://www.cnblogs.com/jycjy/p/6741808.html)