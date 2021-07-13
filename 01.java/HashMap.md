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



