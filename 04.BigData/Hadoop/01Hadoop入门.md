# Hadoop入门

### Hadoop环境搭建

- 系统：CentOS
- 所需软件：jdk、hadoop

jdk配置跳过、Hadoop配置如下：

vim /etc/profile

```
export HADOOP_HOME=/usr/local/hadoop-2.10.1
export PATH=$HADOOP_HOME/bin:$PATH
```

重新刷新配置 source /etc/profile

查看版本  hadoop version



接下来对hadoop进行配置

首先是 core-site.xml

vim /opt/software/hadoop-2.10.1/etc/hadoop/core-site.xml

```
<configuration>
    <property>
        <!-- 这里填的是你自己的ip，端口默认-->
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.182.3:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <!-- 这里填的是你自定义的hadoop工作的目录，端口默认-->
        <value>/usr/local/hadoop-2.9.2/tmp</value>
    </property>
    <property>
    <name>hadoop.native.lib</name>
        <value>false</value>
        <description>Should native hadoop libraries, if present, be used.
        </description>
    </property>
</configuration>
```

接下来是 hadoop-env.sh，配置jdk路径

vim /opt/software/hadoop-2.10.1/etc/hadoop/hadoop-env.sh

![image-20210621164254662](https://youcai922.github.io/src/img/image-20210621164254662.png)

vim /opt/software/hadoop-2.10.1/etc/hadoop/hdfs-site.xml

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.secondary.http.address</name>
        <!--这里是你自己的ip，端口默认-->
        <value>192.168.182.3:50090</value>
    </property>
</configuration>
```

复制默认的cp mapred-site.xml.template ./mapred-site.xml 配置命名为mapred-site.xml

然后编辑vim /opt/software/hadoop-2.10.1/etc/hadoop/mapred-site.xml

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

配置yarn-site.xml

vim /opt/software/hadoop-2.10.1/etc/hadoop/yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <!-- 自己的ip端口默认 -->
        <value>192.168.182.3</value>
    </property>
    <!-- reducer获取数据的方式 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
<!-- Site specific YARN configuration properties -->

</configuration>
```

配置好之后切换到sbin目录下		cd /etc/software/hadoop-2.10.1/sbin

格式化hadoop文件格式，执行命令hadoop namenode -format，成功之后启动

执行启动所有命令 ./start-all.sh

这个时候可以通过jps查看进程

![image-20210621165337937](https://youcai922.github.io/src/img/image-20210621165337937.png)

通过curl localhost50070查看到html，表示启动成功

![image-20210621165401478](https://youcai922.github.io/src/img/image-20210621165401478.png)

如果需要在外面访问虚拟机，则需要关闭防火墙		systemctl start firewalld

防火墙关闭之后：在windows上访问http://192.168.182.3:50070/

出现表示成功访问![image-20210621165138453](https://youcai922.github.io/src/img/\image-20210621165138453.png)
