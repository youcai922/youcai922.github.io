#### scrapy框架：爬虫中封装好的一个明星框架

- 功能：高性能的持久化存储，异步的数据下载，高性能数据解析，分布式

### 使用

##### 环境的安装

- mac or linux：  pip install scrapy

- windows

  ```
  - pip install wheel
  - 下载twisted，下载地址：http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted
  - 安装twisted：pip install Twisted-17.1.0-cp36-cp36m-win_amd64.whl
  - pip install pywin32
  - pip install scrapy
  测试：在终端中输入scrapy命令，没有报错表示安装成功
  ```

##### 创建工程

- 工程目录下scrapy startproject xxxPro
- cd xxxPro
- 在spiders子目录下创建一个爬虫文件
  - scrapy genspider spiderName www.xxx.com

- 执行工程：
  - scrapy crawl spiderName
  - scrapy crawl spiderName --nolog         //不包含输出日志
  - settings.py加上LOG_LEVEL= 'ERROR'   //只输出错误日志