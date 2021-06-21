### Runable和Callable的区别

Runable继承Runable接口，Callable实现Callable接口

runable和callable：call可以抛出异常，有返回值

### 停止线程的方法

stop()已经弃用了，因为会导致数据异常，我们通常使用interrupt()方法来停止线程

### sleep和wait的区别

wait是Object类中的方法，释放了资源，需要用notify和notifyAll唤醒，但是sleep是Threed的方法，不释放资源