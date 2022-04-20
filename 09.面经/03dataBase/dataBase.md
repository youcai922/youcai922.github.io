### Oracle和Mysql的区别

​	mysql默认自增主键，Oracle需要序列
​	Oracle没有limit关键字，需要使用行号分页

#### 优化sql

- 建立索引，优先考虑使用where和group by的字段
- 查询的时候尽量避免使用select * ，返回无用的字段会消耗时间
- 尽量避免使用in和not in，会导致放弃索引而进行全表扫描
- 尽量避免使用or，可以使用union代替
- 尽量避免使用like '%'开头的模糊查询
- 尽量避免在where 左侧条件进行运算、处理和类型转换
- 数据量大的时候避免使用where 1=1
