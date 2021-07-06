# Markdown语法

- 标题
- 字体
- 线
  - [分割线](#title分给线)
  - 删除线
  - 下划线

## 标题

标题使用#做标记，可表示 1-6 级标题，一级标题对应一个 **#** 号，二级标题对应两个 **#** 号，以此类推。

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题



## 字体

markdown的语法子如下

```
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___
```

*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___

## <span id='title分给线'> 分割线</span>

```
***

* * *

*****

- - -

----------
```

***

* * *

*****

- - -

----------



## 删除线

```
RUNOOB.COM
~~BAIDU.COM~~
```

RUNOOB.COM
~~BAIDU.COM~~



## 下划线

```
<u>带下划线文本</u>
```

<u>带下划线文本</u>



## 脚注

```
创建脚注格式类似这样 [^RUNOOB]。

[^RUNOOB]: 这里有脚注
```

创建脚注格式类似这样 [^RUNOOB]。

[^RUNOOB]: 这里有脚注



## 文本间超链接

```
[点我会跳转](#test)
<span id="test">内容</span>
```



表格

```
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
```

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

单元格的合并：

markdown不支持单元格的合并，但是可以通过table标签来进行

```
<table>
	<tr>
	    <th>HBase数据模型</th>
	    <th>java类</th>
	</tr >
	<tr >
        <td rowspan="3">数据库(DataBase)</td>
	    <td>HBaseAdmin</td>
	</tr>
	<tr>
	    <td>HBaseConfiguration</td>
	</tr>
	<tr>
	    <td>HTable</td>
	</tr>
</table>
```

<table>
	<tr>
	    <th>HBase数据模型</th>
	    <th>java类</th>
	</tr >
	<tr >
        <td rowspan="3">数据库(DataBase)</td>
	    <td>HBaseAdmin</td>
	</tr>
	<tr>
	    <td>HBaseConfiguration</td>
	</tr>
	<tr>
	    <td>HTable</td>
	</tr>
</table>





