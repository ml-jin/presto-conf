# Presto - on - yarn SSL 问题解决 (SSLError: Failed to connect. Please check openssl library versions)

[TOC]

## SSLError: Failed to connect. Please check openssl library versions.

### **错误描述：**

在查看agent的日志时，发现报错
SSLError: Failed to connect. Please check openssl library versions.

### 解决方法：

####　1.查看openssl版本：

```
openssl version
1
```

 如果低于 openssl-1.0.1e-16.el6.x86_64 版本，则需要更新到 openssl-1.0.1e-16.el6.x86_64 及以上版本

#### 2. 查看 Python 版本：

```
 python -V
1
```

 如果低于 Python 2.7 版本，则升级 Python 到 2.7 及以上版本。



#### 3.编辑 /etc/python/cert-verification.cfg 配置文件，将 [https] 节的 verify 项设为禁用：

```
vi /etc/python/cert-verification.cfg
1
```

 在配置中更改下列参数

```
[https]
verify=disable
12
```

#### 4.编辑 /etc/ambari-agent/conf/ambari-agent.ini 配置文件，在 [security] 节部分，确保设置如下两个值，其它值保持不变：

```
 vi /etc/ambari-agent/conf/ambari-agent.ini
1
[security]
ssl_verify_cert=0
force_https_protocol=PROTOCOL_TLSv1_2
123
```

#### 5.重启ambari-agent

```
ambari-agent restart
1
```

重新执行，即可通过确认主机并完成注册。