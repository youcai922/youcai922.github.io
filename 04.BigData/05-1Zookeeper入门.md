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

#clientPort=12181	表示连接端口是12181

#server.A=B:C:D		其中A表示这是第几个服务器，B是ip地址，C是数据同步端口号，D是Leader选举端口号

然后在各个zookeeper的根目录下新建data目录，然后分别运行

```
echo '1' > data/myid
echo '2' > data/myid
echo '3' > data/myid
```

切换到bin目录下

```
sh zkServer.sh start	//启动zookeeper
sh zkServer.sh status	//查看当前zookeeper状态，会显示身份
```



### 通过Java连接Zookeeper

方法一：使用zookeeper原始API

```java
ZooKeeper zooKeeper = new ZooKeeper("192.168.182.3:12181,192.168.182.3:12182,192.168.182.3:12183", 5000, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                if(watchedEvent.getState() == Event.KeeperState.SyncConnected){
                    //确认已经连接完毕后再进行操作
                    new CountDownLatch(1).countDown();
                    System.out.println("已经获得了连接");
                }
            }
        });
//连接完成之前先等待
latch.await();
ZooKeeper.States states = zooKeeper.getState();
System.out.println(states);
```

方法二：使用ZkClient客户端

```java
ZkClient zkClient = new ZkClient("192.168.182.3:12181,192.168.182.3:12182,192.168.182.3:12183",40000);
System.out.println(zkClient.getChildren("/"));
```

方法三：使用curator连接

```java
CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.182.3:12181,192.168.182.3:12182,192.168.182.3:12183", 4000, 4000, new ExponentialBackoffRetry(1000, 3));
//启动
client.start();
System.out.println("连接成功！");
Object o = client.getChildren().forPath("/");
System.out.println(o);
```





### 使用原始API操作节点

```java
public class ConnectionDemo implements Watcher{
    private ZooKeeper zookeeper;

    //超时时间
    private static final int SESSION_TIME_OUT = 2000;
    //zookeeper集群ip
    private static String HOST_IP="192.168.182.3:12181,192.168.182.3:12182,192.168.182.3:12183";
    private CountDownLatch countDownLatch = new CountDownLatch(1);
    //ZooKeeper向客户端发送一个Watcher事件通知时，客户端就会对相应的process方法进行回调，从而实现对事件的处理
    @Override
    public void process(WatchedEvent event) {
        if (event.getState() == Event.KeeperState.SyncConnected) {
            System.out.println("Watch received event");
            countDownLatch.countDown();
        }
    }

    //连接zookeeper
    public void connectZookeeper(String host) throws Exception{
        zookeeper = new ZooKeeper(host, SESSION_TIME_OUT, this);
        countDownLatch.await();
        System.out.println("zookeeper connection success");
    }

    // 创建节点
    public String createNode(String path,String data) throws Exception{
        return this.zookeeper.create(path, data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    //获取路径下所有子节点
    public java.util.List<String> getChildren(String path) throws KeeperException, InterruptedException{
        List<String> children = zookeeper.getChildren(path, false);
        return children;
    }

    //获取节点上面的数据
    public String getData(String path) throws KeeperException, InterruptedException{
        byte[] data = zookeeper.getData(path, false, null);
        if (data == null) {
            return "";
        }
        return new String(data);
    }

    // 设置节点信息
    public Stat setData(String path,String data) throws KeeperException, InterruptedException{
        Stat stat = zookeeper.setData(path, data.getBytes(), -1);
        return stat;
    }

    // 删除节点
    public void deleteNode(String path) throws InterruptedException, KeeperException{
        zookeeper.delete(path, -1);
    }

    //获取创建时间
    public String getCTime(String path) throws KeeperException, InterruptedException{
        Stat stat = zookeeper.exists(path, false);
        return String.valueOf(stat.getCtime());
    }

    //获取某个路径下孩子的数量
    public Integer getChildrenNum(String path) throws KeeperException, InterruptedException{
        int childenNum = zookeeper.getChildren(path, false).size();
        return childenNum;
    }
    // 关闭连接
    public void closeConnection() throws InterruptedException{
        if (zookeeper != null) {
            zookeeper.close();
        }
    }

    private static CountDownLatch latch = new CountDownLatch(1);
    public static void main(String[] args) throws Exception {

        ConnectionDemo zookeeper = new ConnectionDemo();
        zookeeper.connectZookeeper(HOST_IP);
//        zookeeper.createNode("/test","ceshi");
        List<String> children = zookeeper.getChildren("/");
        System.out.println(children);
    }
}
```



