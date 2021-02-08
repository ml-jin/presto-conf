IPv4 和 IPV6 用法

[TOC]

## 如何用SSH连接在局域网中的服务器

------

解决这个问题的主要思路有两个，第一个是在路由器上设置NAT，进行端口映射；第二个便是利用IPv6登录。其中，IPv6的方式最方便（Linux默认是开启IPv6服务的），无须多余设置，只需要知道IPv6地址即可。具体方法如下：（假设IPv6地址为2101:da8:a000:12:bc26:9915:4b1d:64cc）

### 1. SSH远程登录服务器

```
# command pattern:
ssh [username]@[IPv6_Host] -p [port number]
# command example:
ssh lg@2101:da8:a000:12:bc26:9915:4b1d:64cc -p 1234
```

为了方便起见（IPv6的地址太长了，很难记住），可以把命令加入到 `.bashrc` 文件中去
`alias log2s="ssh lg@2101:da8:a000:12:bc26:9915:4b1d:64cc -p 1234"`

### 2. SCP拷贝文件

```
# command pattern:
scp [username]@[IPv6_Host]:[file_path] [target_path]
# command example:
scp lg@\[2101:da8:a000:12:bc26:9915:4b1d:64cc\]:/home/lg/example.c ~/home/lg/src
```

这里需要注意的是，由于IPv6地址中的冒号和指定路径之前的冒号有冲突，需要用中括号加转义字符的方式把IPv6的地址括起来。

同样的，也可以将命令的前半段添加到 `.bashrc` 中。