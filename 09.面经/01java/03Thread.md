#### 线程创建方法

- 写一个类继承子Thread类，重写run方法
- 写一个类重写Runable接口，重写run方法
- 写一个类重写Callable接口，重写call方法
- 使用线程池



### Runable和Callable的区别（使用Callable()创建线程比另外两种方式的优势）

Runable继承Runable接口，Callable实现Callable接口

runable和callable：call可以抛出异常，有返回值

### 停止线程的方法

stop()已经弃用了，因为会导致数据异常，我们通常使用interrupt()方法来停止线程

### sleep和wait的区别

- wait是Object类中的方法，释放了资源，需要用notify和notifyAll唤醒，但是sleep是Threed的方法，不释放资源
- sleep方法暂停了执行指定的时间，让出CPU给其他线程，但其监控状态依然保持再指定时间过后又会自动恢复运行状态
- 在调用sleep方法的过程中，线程不会释放对象锁，而wait会释放对象锁

#### wait为什么是Object的方法

所谓的释放锁资源实际是通知对象内置的monitor对象进行释放，而只有所有对象都有内置的monitor对象才能实现任何对象的锁资源都可以释放。又因为所有类都继承自Object，所以wait(）就成了Object方法，也就是通过wait()来通知对象内置的monitor对象释放，而且事实上因为这涉及对硬件底层的操作，所以wait()方法是native方法，底层是用C写的

