#### Mybatis存在哪些优点和缺点

优点：

- 基于SQL语言编程，，相当灵活，不会对应⽤程序或者数据库的现有设计造成任何影响，SQL单独 写，解除 sql 与程序代码的耦合，便于统⼀管理
- 与 JDBC 相⽐，减少了 50%以上的代码量，消除了 JDBC ⼤量冗余的代码，不需要⼿动开关连接
- 很好的与各种数据库兼容（ 因为 MyBatis 使⽤ JDBC 来连接数据库，所以只要JDBC ⽀持的数据 库MyBatis 都⽀持）
- 能够与 Spring 很好的集成；

提供映射标签， ⽀持对象与数据库的 ORM 字段关系映射； 提供对象关系映射标签， ⽀持对象关 系组件维护。
 缺点：

- SQL 语句的编写⼯作量较⼤， 尤其当字段多、关联表多时， 对开发⼈员编写SQL 语句的功底有⼀ 定要求
- SQL 语句依赖于数据库， 导致数据库移植性差， 不能随意更换数据库



#### Mybatis中#{}和${}的区别是什么

- #{}是预编译处理，是占位符，${}是字符串替换，是拼接符
- Mybatis在处理#{}时，就会将sql中的#{}替换成？，调用PreparedStatement来赋值
- Mybatis在处理${}时，就是把${}替换成变量的值，调用Statement来赋值
- 使用#{}可以有效防止SQL注入，提高系统安全性



#### mybtatis一级缓存和二级缓存

- 一级缓存Mybatis的一级缓存是指SQLSession，一级缓存的作用域是SQlSession，Mabits默认开启一级缓存；在同一个SqlSession中，执行相同的SQL查询时；第一次会去查询数据库，并写在缓存中，第二次会直接从缓存中取。 当执行SQL时候两次查询中间发生了增删改的操作，则SQLSession的缓存会被清空。

  每次查询会先去缓存中找，如果找不到，再去数据库查询，然后把结果写到缓存中。 Mybatis的内部缓存使用一个HashMap，key为hashcode+statementId+sql语句。Value为查询出来的结果集映射成的java对象。 SqlSession执行insert、update、delete等操作commit后会清空该SQLSession缓存。

- Mybatis默认是没有开启二级缓存的。二级缓存是mapper级别的。 第一次调用mapper下的SQL去查询用户的信息，查询到的信息会存放代该mapper对应的二级缓存区域。 第二次调用namespace下的mapper映射文件中，相同的sql去查询用户信息，会去对应的二级缓存内取结果。

#### Mybatis 和 Mybatis Plus 的区别
- MyBatis:
  - 所有SQL语句全部自己写
  - 手动解析实体关系映射转换为MyBatis内部对象注入容器
  - 不支持Lambda形式调用
- Mybatis Plus:
  - 强大的条件构造器,满足各类使用需求
  - 内置的Mapper,通用的Service,少量配置即可实现单表大部分CRUD操作
  - 支持Lambda形式调用
  - 提供了基本的CRUD功能,连SQL语句都不需要编写
  - 自动解析实体关系映射转换为MyBatis内部对象注入容器

