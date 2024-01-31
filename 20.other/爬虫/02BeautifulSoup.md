## BeautifulSoup

#### 介绍

正则表达式匹配是适用于所有的编程语言的，而bs4只是python

##### 安装和导入

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

# 方法一：读取html文件
f = open('test.html', 'r')
str = f.read()
f.close()
# 创建BeautifulSoup对象，第一个参数为解析的字符串，第二个参数为解析器
soup = BeautifulSoup(str, 'html.parser')

# 方法二
page_text = response.text
soup = BeatifulSoup(page_text,'lxml')



# 匹配内容，第一个为标签名称，第二个为限定属性，下面表示匹配class为test的img标签
img_list = soup.find_all('img', {'class':'test'})

# 遍历标签 
for img in img_list:
    # 获取img标签的src值
    src = img['src']
    print(src)
```



#### 方法介绍

##### 标签定位

- soup.tagName:返回第一次出现的tagName标签名称

  ```python
  soup.a
  soup.div
  ```

- soup.find

  ```python
  soup.find('div')	//等同于soup.div
  soup.find('div',class_/id/attr='song')   //div中class为song的标签
  ```

- soup.find_all

  ```python
  soup.find_all('a',class_/id/attr='song')	//返回符合要求的所有标签，为列表，也可以添加条件
  ```

- soup.select

  ```python
  soup.select('.tang')		//根据选择器（id、class、层级选择器）去选择，最后返回的是列表
  soup.select('.tang > ul > li > a')[0]  //空格表示多个层级，>表示一个层级
  ```

##### 属性值获取

- soup.a.text/string/get_text()标签之间的文本数据

  ```python
  soup.a.text/string/get_text()
  # soup.text/soup.get_text()可以获取某个标签中所有的文本内容
  # soup.string只可以获取该标签下面直系的文本内容
  ```

- 标签的属性

  ```
  soup.a['herf']
  ```



## XPath解析

#### xpath表达式

- /：表示的是从根节点开始定位。标识的是一个层级
- //：表示的是多个层级。可以表示从任意位置开始定位
- 属性定位： //div[@class='song']     tag[@attrName="attrValue"]
- 索引定位：//div[@class="song"]/p[3]           索引是从1开始的
- 取文本：
  - /text()		取标签中直系的文本内容
  - //text()       取标签中非直系的文本内容（所有文本内容）
- 取属性
  - @attrName

```python
# 知乎的热榜是ajax动态生成的，这里只做代码爬取效果展示，功能未实现
import requests
from lxml import etree

if __name__ == "__main__":
    url = 'https://www.zhihu.com/hot'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
    }
    zhiHuHot_text = requests.get(url=url, headers=headers).text

    print(zhiHuHot_text)
    tree = etree.HTML(zhiHuHot_text)
    hot_list = tree.xpath('//div[@class="HotList-list"]/section[@class="HotItem"]')
    for hot_detail in hot_list:
        title = hot_detail.xpath('./div[@class="HotItem-content"]/a/@title')
```

```python
# 爬取某小说网站的热门下载
import requests
from lxml import etree

if __name__ == "__main__":
    url = 'https://www.txt80.cc/hot/'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
    }
    response = requests.get(url=url, headers=headers)
    response.encoding = 'utf-8'
    tree = etree.HTML(response.text)
    hot_list = tree.xpath('//div[@class="list_l_box"]/div[@class="slist"]')
    for item in hot_list:
        title = item.xpath('./div[@class="info"]/h4/a/text()')[0]
        print(title)
```

```python
# 拉取优衣库商品价格信息
import requests

if __name__ == "__main__":
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
    }
    total_url = 'https://www.uniqlo.cn/public/bin/data/json/L4JsonData.json'
    response = requests.get(url=total_url, headers=headers)
    product_map = response.json()['CustomerHeart'][0]
    product_info_url_template = 'https://www.uniqlo.cn/data/products/spu/zh_CN/{product_id}.json'
    detail_url_template = 'https://d.uniqlo.cn/p/product/i/product/spu/pc/query/{product_id}/zh_CN'

    file = open("youyiku_info.txt", "w", encoding="utf-8")

    for key, value in product_map.items():
        product_info_url = product_info_url_template.format(product_id=value)
        response = requests.get(url=product_info_url, headers=headers).json()
        product_info_rows = response["rows"]
        product_info_summary = response["summary"]
        product_info_summary_fullName = product_info_summary["fullName"]
        file.write("货号:%s\t对应的产品编号:%s\t商品名称：%s\n" % (key, value, product_info_summary_fullName))

        detail_url = detail_url_template.format(product_id=value)
        product_detail_info = requests.get(url=detail_url, headers=headers).json()
        try:
            product_detail_info_rows = product_detail_info["resp"][0]["rows"]
        except:
            product_detail_info_rows = []
        for sizeinfo in product_info_rows:
            result = list(filter(lambda row: row['productId'] == sizeinfo["productId"], product_detail_info_rows))
            if not result:
                file.write("\t子产品编号：%s\t款式：%s\t大小：%s\n" % (
                    sizeinfo["productId"], sizeinfo["style"], sizeinfo["sizeText"]))
            else:
                file.write("\t子产品编号：%s\t款式：%s\t大小：%s\t价格：%s\n" % (
                    sizeinfo["productId"], sizeinfo["style"], sizeinfo["sizeText"], result[0]["price"]))
    file.close()
    print("同步消息结束")
```

