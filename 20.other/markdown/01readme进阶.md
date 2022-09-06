# 徽标

官网地址：[shields.io](https://shields.io/category/other)

#### 静态徽标

使用https://img.shields.io/badge/<LABEL>-<MESSAGE>-<COLOR>的格式利用img进行引用

```
![](https://img.shields.io/badge/码龄-3年-blue)则会展示如下徽章
```

当然你也可以使用官方给的静态徽标生成器

![](https://youcai922.github.io/99.src/img/shields静态徽标生成器.png)

![](https://img.shields.io/badge/码龄-3年-blue)

- COLOR的类型有：

  - brightgreen![](https://img.shields.io/badge/码龄-3年-brightgreen)

  - green![](https://img.shields.io/badge/码龄-3年-green)

  -  yellowgreen![](https://img.shields.io/badge/码龄-3年-yellowgreen)
  - yellow![](https://img.shields.io/badge/码龄-3年-yellow)

  - orange![](https://img.shields.io/badge/码龄-3年-orange)

  - red![](https://img.shields.io/badge/码龄-3年-red)

  - lightgrey![](https://img.shields.io/badge/码龄-3年-lightgrey)

  - blue![](https://img.shields.io/badge/码龄-3年-blue)
  
  - 当然也支持颜色编码例如：da282a![](https://img.shields.io/badge/码龄-3年-da282a)

#### 动态徽标

![油菜](https://img.shields.io/badge/dynamic/json?color=FE7398&label=bilibili&logo=bilibili&prefix=%E7%B2%89%E4%B8%9D%E6%95%B0%3A&query=%24.data.totalSubs&url=https%3A%2F%2Fapi.spencerwoo.com%2Fsubstats%2F%3Fsource%3Dbilibili%26queryKey%3D251889727)

推荐使用Substats获取数据，然后利用shields.io制作牌子。Substats相关信息：[开源地址](https://github.com/spencerwooo/Substats)、[API文档](https://api.spencerwoo.com/substats/)、[说明文档](https://substats.spencerwoo.com/)

- 根据说明文档，找到对应的API请求

  ![](https://youcai922.github.io/99.src/img/Substats说明文档bilibili演示.png)

  我的B站UID是251889727，所以，请求应该是https://api.spencerwoo.com/substats/?source=bilibili&queryKey=251889727

  通过浏览器请求可以看到已经获取到了粉丝数量

  然后再根据shields.io制作动态的徽标

  ![](https://youcai922.github.io/99.src/img/shields动态徽标生成器.png)

  - 数据类型选JSON
  - label是徽标左侧的标签
  - API地址是刚刚我们的请求
  - query填入返回json的字段
  - color和上面一样，可以有十六进制的颜色代码，或者是枚举类
  - prefix为前缀和后缀

- 美化

  - 添加Logo，URL中加入&logo=bilibili则可以获取到对应的Logo，查找不同的图标可以在[Simpleicons](https://simpleicons.org/)搜索查找
  - 自定义logo，如果需要使用库中没有的logo，则需要自己找到一个图标的svg图标文件，然后转成base64格式，可以在如下网站进行转换https://www.css-js.com/tools/base64.html
  - 取消勾选  使用在css中，然后点击copy code
  - 然后在&logo= 后缀加上刚刚复制的编码，就可以看到自定义的图标了

![](https://youcai922.github.io/99.src/img/图标转svg.png)

- 徽标超链接

  现在我们拥有了徽标，但是徽标点击不能够跳转到我们想要的网站，我们可以在徽标外嵌套一个超链接达到效果

  [![油菜](https://img.shields.io/badge/dynamic/json?color=FE7398&label=bilibili&prefix=%E7%B2%89%E4%B8%9D%E6%95%B0%3A&query=%24.data.totalSubs&url=https%3A%2F%2Fapi.spencerwoo.com%2Fsubstats%2F%3Fsource%3Dbilibili%26queryKey%3D251889727)](https://space.bilibili.com/251889727)

  ```
  [![油菜](https://img.shields.io/badge/dynamic/json?color=FE7398&label=bilibili&prefix=%E7%B2%89%E4%B8%9D%E6%95%B0%3A&query=%24.data.totalSubs&url=https%3A%2F%2Fapi.spencerwoo.com%2Fsubstats%2F%3Fsource%3Dbilibili%26queryKey%3D251889727)](https://space.bilibili.com/251889727)
  ```

# 表情

```
:walking:      的格式在markdown中可以显示表情
```



