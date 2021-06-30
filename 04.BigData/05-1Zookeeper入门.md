zookeeper安装见Linux工具安装

#### 伪集群模式搭建：

复制已经搭建好的单继zookeeper

修改conf.cfg文件

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/apache-zookeeper-3.7.0-bin1/data
dataLogDir=/opt/software/apache-zookeeper-3.7.0-bin1/logs
clientPort=12181
server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/apache-zookeeper-3.7.0-bin2/data
dataLogDir=/opt/software/apache-zookeeper-3.7.0-bin2/logs
clientPort=12182
server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/apache-zookeeper-3.7.0-bin/data
dataLogDir=/opt/software/apache-zookeeper-3.7.0-bin/logs
clientPort=12181
server.1=127.0.0.1:12888:13888
server.2=127.0.0.1:14888:15888
server.3=127.0.0.1:16888:17888
```

