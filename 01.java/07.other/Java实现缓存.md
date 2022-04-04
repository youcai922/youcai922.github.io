- 定义缓存封装类

```java
@ToString
public class EntityCache {
    //保存的数据
    private Object datas;

    //设置数据失效时间,为0表示永不失效
    private long timeOut;

    //最后刷新时间
    private long lastRefeshTime;

    public EntityCache(Object datas, long timeOut, long lastRefeshTime) {
        this.datas = datas;
        this.timeOut = timeOut;
        this.lastRefeshTime = lastRefeshTime;
    }

    public EntityCache(Object datas) {
        this.datas = datas;
        this.timeOut = 0L;
        this.lastRefeshTime = 0L;
    }

    public Object getDatas() {
        return datas;
    }
    
    public void setDatas(Object datas) {
        this.datas = datas;
    }
    
    public long getTimeOut() {
        return timeOut;
    }
    
    public void setTimeOut(long timeOut) {
        this.timeOut = timeOut;
    }
    
    public long getLastRefeshTime() {
        return lastRefeshTime;
    }
    
    public void setLastRefeshTime(long lastRefeshTime) {
        this.lastRefeshTime = lastRefeshTime;
    }
}
```

- 编写工具类

```java
public class CacheUtil{
    private static Map<String, EntityCache> caches = new ConcurrentHashMap<String, EntityCache>();

    //存入缓存
    public static void putCache(String key, EntityCache cache) {
        caches.put(key, cache);
    }

    //存入缓存
    public static void putCache(String key, Object datas, long timeOut) {
        timeOut = timeOut > 0 ? timeOut : 0L;
        putCache(key, new EntityCache(datas, timeOut, System.currentTimeMillis()));
    }

    //获取对应缓存
    public static EntityCache getCacheByKey(String key) {
        if (CacheUtil.isContains(key)) {
            return caches.get(key);
        }
        return null;
    }

    //获取对应缓存
    public static Object getCacheDataByKey(String key) {
        if (CacheUtil.isContains(key)) {
            return caches.get(key).getDatas();
        }
        return null;
    }

    //获取所有缓存
    public static Map<String, EntityCache> getCacheAll() {
        return caches;
    }

    //判断是否在缓存中
    public static boolean isContains(String key) {
        return caches.containsKey(key);
    }

    //清除所有缓存
    public static void clearAll() {
        caches.clear();
    }

    // 清除对应缓存
    public static void clearByKey(String key) {
        if (CacheUtil.isContains(key)) {
            caches.remove(key);
        }
    }
    // 缓存是否超时失效
    public static boolean isTimeOut(String key) {
        if (!caches.containsKey(key)) {
            return true;
        }
        EntityCache cache = caches.get(key);
        long timeOut = cache.getTimeOut();
        long lastRefreshTime = cache.getLastRefeshTime();
        if (timeOut == 0 || System.currentTimeMillis() - lastRefreshTime >= timeOut) {
            return true;
        }
        return false;
    }
    
    // 获取所有key
    public static Set<String> getAllKeys() {
        return caches.keySet();
    }
}
```



- 使用工具类

```java
@Service
public class PropertiesUtil {

    private static PropertyMapper propertyMapper;

    //这里利用set注入，不然propertyMapper会报nullException
    @Autowired
    public void setPropertyMapper(PropertyMapper propertyMapper){
        PropertiesUtil.propertyMapper = propertyMapper;
    }

    public static Map<String,Object> getPropertyMap(String propertyType){
        List<Map<String,Object>> tempList = propertyMapper.queryPropertyInfo(propertyType);
        Map<String,Object> resultMap = new HashMap<>();
        tempList.stream().forEach(item->{
            resultMap.put(item.get("propertyCode").toString(),item.get("propertyValue"));
        });
        return resultMap;
    }
}

public class SFTP {
    public void connect() throws Exception {
        //如果缓存中没有则取数据库中读取
        if (!CacheUtil.isContains("SFTPCache")){
			CacheUtil.putCache("SFTPCache",new EntityCache(new PropertiesUtil().getPropertyMap("SFTP")));
		}
        
        ip = String.valueOf(((Map) CacheUtil.getCacheByKey("SFTPCache").getDatas()).get("ftp.ip"));
        //获取sftp连接.....
    }
}
```

