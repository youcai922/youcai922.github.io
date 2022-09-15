#### 动态set方法

```java
private void setValue(Object dto, String name, Object value) {
    try {
        Method[] m = dto.getClass().getMethods();
        for (int i = 0; i < m.length; i++) {
            if (("set" + name).toLowerCase().equals(m[i].getName().toLowerCase())) {
                m[i].invoke(dto, value);
                break;
            }
        }
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}
```

#### 动态get方法

```java
private static String getValue(Object dto, String name) {
    try {
        Method m = (Method) dto.getClass().getMethod(("get" + name));
        String val = (String) m.invoke(dto);// 调用getter方法获取属性值
        return val;
    } catch (NoSuchMethodException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    }
    return null;
}
```

