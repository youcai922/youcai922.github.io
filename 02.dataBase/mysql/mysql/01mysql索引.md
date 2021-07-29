## 索引：

### Mysql索引主要有两种数据结构：B+Tree和Hash索引

- B+Tree索引

  是mysql使用最频繁的一个索引数据结构，是Inodb和myisam存储引擎模式的索引类型。相对Hash索引 ，B+Tree在查找单跳记录的速度比不上hash索引，但是因为更适合排序等操作，索引他更受欢迎。毕竟不可能只对数据库进行单条记录的操纵

- Hash索引

  mysql中，只有memory（memory表只存在内存中，断电会消失，适用于临时表）存储引擎显示支持Hash索引，是Memory表的默认索引类型，尽管Memory表也可以使用B+Tree索引。Hash索引把数据以hash形式组织起来，因此当查找某一条记录的时候，速度非常快。但是因为hash结构，每个键只对应一个值，而且是散列的方式分布。所以它不支持范围查找和排序等功能。



### 常见的索引类型

主键索引、唯一索引、普通索引、全文索引、组合索引

- 普通索引（INDEX）：基本的索引，没有任何限制

```sql
ALTER TABLE 'table_name' ADD INDEX index_name('col')
```

- 唯一索引：索引列的值必须唯一，但是允许有空值

```sql
ALTER TABLE'table_name'ADD UNIQUE('col')
```

- 主键索引：特殊的唯一索引，不允许有空值

```sql
ALTER TABLE 'table_name' ADD PRIMARY KEY('col')
```

- 全文索引：仅可用于MyISAM和InoDB，对于较大的数据，生成全文索引很消耗空间

```sql
ALTER TABLE 'table_name' ADD FULLTEXT('col')
```

#### 组合索引：

```sql
ALTER TABLE 'table_name' ADD INDEX index_name('col1','col2','col3')
```

为了更多的提高mysql效率可以建立组合索引，遵循“最左前缀”原则。创建符合索引应该将最常用（频率）作为限制条件的列放在最左边，依次递减。组合索引最左字段用in是可以用到索引的。相当于建立了col1,col1col2,col1col2col3三个索引。

最左前缀匹配原则：mysql会一直向右匹配直到遇到范围查询（>,<,between,like）就会停止匹配。

如果建立（a,b,c,d）顺序索引，d是用不到索引的，如果建立（a,b,c,d）顺序索引则都可以用到，abd顺序是可以任意调整的。

in和=可以乱序，比如a=1 and b=2 and c=3建立（a,b,c）索引可以任意顺序，mysql查询优化器可以帮你申城执行计划，使得索引可以被识别的形式。

### 注意事项

- 不要滥用索引

  - 索引提高查询速度，却会降低更新表的速度，因为更新表的时候，mysql不仅要更新数据，保存数据，还要更新索引，保存索引
  - 索引会占用磁盘空间

- 索引不会包含含有NULL值的列

  复合索引只要有一列含有NULL值，那么这一列对于此复合索引就是无效的，因此我们在设计数据库设计时，不要让字段的默认值为NULL

- Mysql查询只是用一个索引

  如果where子句中使用了索引的话，那么order by中的列是不会使用索引的

- like

  like '%aaa%'不会使用索引 而like 'aaa%'会使用索引 