# HashMap

#### 基础特性：

散列表，K-V对，利用HashCode值存储数据，只允许一条记录的k为null，不支持线程同步

无序，集成AbstractMap，实现了Map，Cloneable，java.io.Serializable接口

java.util包

- 常用方法：添加put()，获取元素get()，删除remove()，计算大小size()，克隆clone()，判空isEmpty()，清空clear()

#### HashMap底层：

jdk1.7之前，HashMap是数组+链表实现的，并且采用头插法

jdk1.8之后，HashMap初始采用数组+链表，当链表长度大与阈值（默认为8），则会将链表转换成红黑树，以减少搜索时间。采用尾插法

### HashMap扩容：

HashMap初始大小为16。当HashMap中的元素个数之和大与 **负载因子*当前容量** 的时候就要进行扩容，容量为原来的2倍。

##### HashMap线程是否安全：

- 在jdk1.7中，在多线程环境下，扩容会造成环形链或者数据丢失
- 在jdk1.8中，多线程环境下，会发生数据覆盖的情况

#### HashMap遍历

- 迭代器（Iterator）方法遍历
- foreach
- Lambda表达式
- Stream流

##### 初始化数据

```java
Map<Integer,String> map=new HashMap();
map.put(1,"张三");
map.put(2,"李四");
map.put(3,"王五");
map.put(4,"赵六");
map.put(3,"宋七");
```

##### 1.迭代器EntrySet

```java
Iterator<Map.Entry<Integer,String>> iterator = map.entrySet().iterator();
while(iterator.hashNext()){
    Map.Entry<Integer,String> entry = iterator.next();
    System.out.print(entry.getKey());
    System.out.print(entry.getValue());
}
```

##### 2.迭代器KeySet

```java 
Iterator<Map.Entry<Integer,String>> iterator = map.keySet().iterator();
while(iterator.hashNext()){
    Map.Entry<Integer,String> entry = iterator.next();
    System.out.print(entry.getKey());
    System.out.print(entry.getValue());
}
```

##### 3.ForEach EntrySet

```java
for(Map.Entry<Integer,String> entry : map.entrySet()){
	System.out.print(entry.getKey());
	System.out.print(entry.getValue());
}
```

##### 4.ForEach KeySet

```Java
for(Integer key : map.keySet()){
	System.out.print(key);
	System.out.print(map.get(key));
}
```

​	entrySet迭代具有轻微的性能优势(大约快10%)

##### 5.Lambda

```java
map.forEach((key, value) -> {
    System.out.print(key);
    System.out.print(value);
});
```

##### 6.使用Iterator迭代

//使用泛型

```java
Map<Integer,Integer> map = new HashMap<Integer,Integer>();
Iterator<Map.Entry<Integer,Integer>> entries = map.entrySet().iterator();
while(entries.hashNext()){
    Map.Entry<Integer,Integer> entry = entries.next();
    System.out.println("key="+entry.getKey()+",Value="+entry.getValue(0));
}
```

//不使用泛型

```java
Map map=new HashMap();
Iterator entries = map.entrySet().iterator();
while(entries.hasNext()){
    Map.Entry entry=(Map.Entry) entries.next();
    Integer key = (Integer)entry.getKey();
    Integer value = (Integer)entry.getValue();
    System.out.println("Key = " + key + ", Value = " + value);
}
```

##### 7.迭代keys并搜索values（低效的）

```java
Map<Integer,Integer> map = new HashMap<Integer,Integer>();
for(Integer key : map.keySet()){
    Integer value = map.get(key);
    System.out.println("key = "+key+",Value = "+ value);
}
```

​	这个方法看上去比方法1更简洁，但是实际上它更慢更低效，通过key得到value值更耗时（这个方法在所有实现map接口的map中比方法1慢20%-200%。

##### 8.Stream流

```Java
map.entrySet().stream().forEach((entry) -> {
    System.out.print(entry.getKey());
    System.out.print(entry.getValue());
});
```



