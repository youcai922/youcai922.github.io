# Hadoop入门

Hadoop一共有三种模式：单机模式、伪集群模式和集群模式

hadoop下载地址：https://hadoop.apache.org/releases.html

### Hadoop单机模式搭建

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
        <value>/usr/local/hadoop-2.10.1/tmp</value>
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

![image-20210621164254662](https://youcai922.github.io/99.src/img/image-20210621164254662.png)

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

![image-20210621165337937](https://youcai922.github.io/99.src/img/image-20210621165337937.png)

通过curl localhost50070查看到html，表示启动成功

![image-20210621165401478](https://youcai922.github.io/99.src/img/image-20210621165401478.png)

如果需要在外面访问虚拟机，则需要关闭防火墙		systemctl stop firewalld

防火墙关闭之后：在windows上访问http://192.168.182.3:50070/

出现表示成功访问![image-20210621165138453](https://youcai922.github.io/99.src/img/image-20210621165138453.png)

hadoop管理界面：8088端口

NameNode界面：50070端口

HDFS NameNode界面：8042端口



### hadoop命令

```
hdfs dfs -help							查看帮助
hdfs dfs -ls /							查看当前目录信息
hdfs dfs -put /本地路径 /hdfs路径			上传文件
hdfs dfs -moveFromLocal a.txt /aa.txt	剪切文件
hdfs dfs -get /hdfs路径 /本地路径			下载文件到本地
hdfs dfs -getmerge /hdfs路径文件夹 /合并后的文件	合并下载
hdfs dfs -mkdir /hello					创建文件夹
hdfs dfs -mkdir -p /hello/world			创建多级文件夹
hdfs dfs -mv /hdfs路径 /hdfs路径		 移动hdfs文件夹
hdfs dfs -cp /hdfs路径 /hdfs路径		 复制hdfs文件夹
hdfs dfs -rm /aa.txt					删除hdfs文件
hdfs dfs -rm -r /hello					删除hdfs文件夹
hdfs dfs -cat /文件					  查看hdfs文件
hdfs dfs -tail -f /文件				  查看hdfs文件的最后几行
hdfs dfs -count /文件夹				  查看文件夹中有多少个文件
hdfs dfs -df /							查看hdfs总空间
hdfs dfs -df -h /						查看hdfs总空间
hdfs dfs -setrep 1 /a.txt				修改副本数量
```

### 文件上传，下载和读取

创建Maven工程

```
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>3.1.4</version>
    </dependency>
</dependencies>
```

**文件上传**

```
try {
    Configuration conf = new Configuration();

    //这里指定使用的是 hdfs文件系统
    conf.set("fs.defaultFS", "hdfs://192.168.182.3:9000/hello");

    //通过这种方式设置java客户端身份
    System.setProperty("HADOOP_USER_NAME", "root");
    FileSystem fs = null;

    fs = FileSystem.get(conf);

    //或者使用下面的方式设置客户端身份
    //FileSystem fs = FileSystem.get(new URI("hdfs://master:9000"),conf,"root");

    // fs.create(new Path("/helloByJava")); //创建一个目录

    FSDataOutputStream outputStream = fs.create(new Path("/2.txt"), true); //输出流到HDFS
    FileInputStream inputStream = new FileInputStream("D:/下载/2.txt"); //从本地输入流。
    IOUtils.copy(inputStream, outputStream); //完成从本地上传文件到hdfs

    fs.close();
} catch (IOException e) {
	e.printStackTrace();
}
```

**文件下载**

```
try {
    Configuration conf = new Configuration();

    //这里指定使用的是 hdfs文件系统
    conf.set("fs.defaultFS", "hdfs://192.168.182.3:9000/hello");

    //通过这种方式设置java客户端身份
    System.setProperty("HADOOP_USER_NAME", "root");
    FileSystem fs = null;

    fs = FileSystem.get(conf);

    //或者使用下面的方式设置客户端身份
    //FileSystem fs = FileSystem.get(new URI("hdfs://master:9000"),conf,"root");

    // fs.create(new Path("/helloByJava")); //创建一个目录

    //文件下载到本地 如果出现0644错误或找不到winutils.exe,则需要设置windows环境和相关文件.
    fs.copyToLocalFile(new Path("/2.txt"), new Path("D:\\下载"));

    fs.close();
} catch (IOException e) {
	e.printStackTrace();
}
```

**文件读取**

```
try {
    // 配置连接地址
    Configuration conf = new Configuration();
    conf.set("fs.defaultFS", "hdfs://192.168.182.3:9000");
    FileSystem fs = FileSystem.get(conf);
    // 打开文件并读取输出
    Path hello = new Path("/2.txt");
    FSDataInputStream ins = fs.open(hello);
    int ch = ins.read();
    while (ch != -1) {
    System.out.print((char)ch);
    ch = ins.read();
}
	System.out.println();
} catch (IOException ioe) {
	ioe.printStackTrace();
}
```



