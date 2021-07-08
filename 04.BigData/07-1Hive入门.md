#### Hive基本操作

```
启动Hive
	bin/hive
查看数据库
	hive> show databases;
打开默认数据库
	hive> use default;
显示default数据库中的表
	hive> show tables;
创建一张表
	hive> create table student(id int,name String);
显示数据库中有几张表
	hive> show tables;
查看表的结构
	hive> desc student
向表中插入数据
	hive> insert into student values(1000,"ss");
查询表中数据
	hive> select * from student
退出hive
	hive> quit;
```

#### 将本地文件导入Hive案例

- 数据准备

在opt/software/datas目录下准备数据

```
创建数据目录datas
    cd opt/software
    mkdir datas

创建数据源student.txt
	touch student.txt
	vim student.txt
1001	zhangsan
1002	lisi
1003	wangwu
```

- 实际操作：

```sql
启动hive
	bin/hive
展示数据库
	hive> show databses;
使用default数据库
	hive> use default;
显示default数据库中的表
	hive> show default;
删除已创建的student表
	hive> drop table student;
创建student表，并声明文件分隔符为‘\t’
	hive>  create table student(id int,name String)ROW FORMAT DELIMTED FIELDS TERM NATED BY '\t'
加载/opt/software/datas/student.txt文件到student数据库表中
	hive> load data local inpath '/opt/software/datas/students.txt' into table student;
hive查询结果
	hive> select * from student;
```





