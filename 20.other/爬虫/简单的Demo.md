### 爬取一个简单的网页

```python
import requests
# 以get方法发送请求，返回数据
response = requests.get('http://www.baidu.com')
# 以二进制写入的方式打开一个文件
f = open('index.html', 'wb')
# 将响应的字节流写入文件
f.write(response.content)
# 关闭文件
f.close()
```



### 爬取网页中的图片

```python
import requests
# 准备url
url = 'https://img-blog.csdnimg.cn/2020051614361339.jpg'
# 发送get请求
response = requests.get(url)
# 以二进制写入的方式打开图片文件
f = open('test.jpg', 'wb')
# 将文件流写入图片
f.write(response.content)
# 关闭文件
f.close()
```





## BeautifulSoup

安装和导入

```cmd
#安装
pip install beautifulsoup4
#模块的导入
from bs4 import BeautifulSoup
```

### 使用Demo

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <img class="test" src="1.jpg">
    <img class="test" src="2.jpg">
    <img class="test" src="3.jpg">
    <img class="test" src="4.jpg">
    <img class="test" src="5.jpg">
    <img class="test" src="6.jpg">
    <img class="test" src="7.jpg">
    <img class="test" src="8.jpg">
</body>
</html>
```

```python
from bs4 import BeautifulSoup

# 读取html文件
f = open('test.html', 'r')
str = f.read()
f.close()

# 创建BeautifulSoup对象，第一个参数为解析的字符串，第二个参数为解析器
soup = BeautifulSoup(str, 'html.parser')

# 匹配内容，第一个为标签名称，第二个为限定属性，下面表示匹配class为test的img标签
img_list = soup.find_all('img', {'class':'test'})

# 遍历标签 
for img in img_list:
    # 获取img标签的src值
    src = img['src']
    print(src)
```

