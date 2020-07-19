# Scrapy框架

### Scrapy简介

Scrapy文档参看：[Scrapy官方文档](https://doc.scrapy.org/en/latest/intro/overview.html#)  

##### Scrapy安装

Scrapy是一个完整的爬虫框架，在安装过程中会涉及到许多依赖库，需要一步一步安装：

**使用Python**

首先提醒：**有的库是直接程序就可以安装，有的库需要下载下来手动安装。**

手动安装库的名称通常是下面这种格式，选择适合自己环境的版本下载安装：

```
库名-18.9.0-cp36-cp36m-win_amd64
    18.9.0：库的版本号
    cp36：适合python3.6
    win_amd64：适合windows64位
```

1. 安装wheel库：命令行中输入`pip install wheel`命令进行安装。
2. 安装lxml库：在 [http://www.lfd.uci.edu/~gohlke/pythonlibs/](http://www.lfd.uci.edu/~gohlke/pythonlibs/) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
3. 安装zope.interface库：在 [https://pypi.python.org/pypi/zope.interface#downloads](https://pypi.python.org/pypi/zope.interface#downloads) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
4. 安装pyOpenssl库：在 [https: //pypi python.org pypi pyOpenSSL#downloads](https: //pypi python.org pypi pyOpenSSL#downloads) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
5. 安装Twisted库：在 [http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted](http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
6. 安装pywin32库：在 [https://sourceforge.net/projects/pywin32/files/pywin32/Build%20220/](https://sourceforge.net/projects/pywin32/files/pywin32/Build 220/) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。但是容易出现问题，就是在安装时会出现提示python3.6-32在注册表中不存在。首先根据(http://blog.csdn.net/u014680513/article/details/51005650)把Python写入注册表，在HKEY-CURRENT_USER–Software–Python–PythonCore–会发现3.6文件夹，把这个文件夹导出并重命名为3.6-32，然后导入注册表。就可以安装Pywin32了。
7. 安装Scrapy框架：命令行中输入`pip install scrapy`命令进行安装。

**使用Anaconda**











**Scrapy 是一个基于 Twisted 的多线程异步处理框架，所以scrapy是自带多线程的。**

另外，**Scrapy也是异步的，是基于Twisted事件驱动的**。在任何情况下，都不要写阻塞的代码。例如：

1. 访问文件、数据库或者Web

2. 产生新的进程并需要处理新进程的输出，如运行shell命令

3. 执行系统层次操作的代码，如等待系统队列

优点：架构清晰， 模块之间的耦合程度低（相互影响小），可扩展性极强，可以灵活完成各种需求，我们只需要定制开发几个模块就可以轻实现一个爬虫。

**Scrapy数据流**

![img](https://upload-images.jianshu.io/upload_images/1370011-a3be891e6a48dad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/706/format/webp)

1、引擎打开起点网站(open a domain)，向处理该网站的Spider请求第⼀个要爬取的URL。

2、引擎从Spider中获取到第⼀个要爬取的URL并在调度器(Scheduler)调度。

3、引擎向调度器请求下⼀个要爬取的URL。

4、调度器返回下⼀个要爬取的URL给引擎，引擎将URL通过下载中间件转发给下载器(Downloader)。

5、页面下载完毕，下载器生成页面的Response，并将其通过下载中间件发送给引擎。

6、引擎从下载器中接收到Response并通过Spider中间件发送给Spider处理。

7、Spider在Response中爬取到的Item和新的Request给引擎。

8、引擎将爬取到的Item给Item Pipeline，将Request给调度器。

9、(从第⼆步)重复直到调度器中没有更多地request，引擎关闭该⽹站。

# Scrapy项目

### 创建项目

在要创建项目的文件下打开命令行输入：

```
scrapy  startproject  test1（项目名称）
```

这时在目标文件就生成了一个test1项目文件夹

##### 项目结构

展开test1项目文件夹下面的所有文件夹，可以看到其全部的项目结构。

```
test1
	-test1
		-spiders
			__init__.py
		__init__.py
		items.py
		middlewares.py
		pipelines.py
		settings.py
	scrapy.cfg			
```

spiders：包含 Spider 文件的文件夹，Spider 文件会在下面的命令中创建。

items.py：它定义 Item 数据结构。

middlewares.py：它定义 Spider Middlewares 和 Downloader Middlewarese的实现。

pipelines.py：它定义 Item Pipeline 的实现。

settings.py：它定义项目的全局配置。

scrapy.cfg：它是 Scrapy 项目的配置文件，其内定义了项目的配置文件路径、部署相关信息等内容。

### settings文件

**禁用robots**

Robots协议（也称为爬虫协议、机器人协议等）的全称是“网络爬虫排除标准”（Robots Exclusion Protocol），网站通过Robots协议告诉爬虫和搜索引擎哪些页面可以抓取，哪些页面不能抓取。防止被爬⽹站的robots.txt起作用，需要关闭Robot协议检查：

```
ROBOTSTXT_OBEY = False
```

**设置请求头**

方法一：取消DEFAULT_REQUEST_HEADERS(默认请求头)的注释并设置请求头内容，然后取消DOWNLOADER_MIDDLEWARES注释使之生效。

```
from fake_useragent import UserAgent

# Override the default request headers:
DEFAULT_REQUEST_HEADERS = {
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
  'Accept-Language': 'en',
  'User-Agent':UserAgent().random
}
```

方法二：取消USER_AGENT注释，添加随机请求头，然后取消DOWNLOADER_MIDDLEWARES注释使之生效。

```
from fake_useragent import UserAgent

USER_AGENT = UserAgent().random
```

**设置MySQL数据库**

```
MYSQL_HOST = '127.0.0.1'
MYSQL_DATABASE = 'spider'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '123456'
MYSQL_PORT = 3306
```

定义MongoDB的连接信息：

```
# 取消ITEM_PIPELINES注释，添加'test1.pipelines.MysqlPipeline': 300。
#  test1项目名称，pipelines文件名称，MysqlPipeline定义的类（在后面按这个名称定义）
# 后面对应的键值代表调用优先级，数字越小越先被调用。
ITEM_PIPELINES = {
   'test1.pipelines.MysqlPipeline': 400,
}
```

**设置MongoDB数据库**

```
# MONGO_URI数据库地址，MONGO_DB数据库名称
MONGO_URI = '127.0.0.1'
MONGO_DB = '...'
```

定义MongoDB的连接信息：

```
# 取消ITEM_PIPELINES注释，添加'test1.pipelines.MongoPipeline': 300。
#  test1项目名称，pipelines文件名称，MongoPipeline定义的类（在后面按这个名称定义）
# 后面对应的键值代表调用优先级，数字越小越先被调用。
ITEM_PIPELINES = {
   'test1.pipelines.MongoPipeline': 400,
}
```

### pipelines文件

Item Pipeline 为项目管道，Item 生成后，它会自动被送到 Item Pipeline 进行处理，将结果保存到数据库，或者筛选某些有用的 Item。

实现 Item Pipeline 需要定义一个类并实现 process_item()方法。该方法有两个参数，一个参数是 item，每次 Spider 生成的 Item 都会作为参数传递过来。另一个参数是 spider，就是 Spider 实例。

启用 Item Pipeline 后， Item Pipeline 会自动调用这个方法， process_item()方法返回包含数据的字典或 Item 对象，或者抛出 Dropltem 异常。

##### 保存MySQL数据库

在pipelines.py 文件添加一个类MysqlPipeline(在上面的设置MySQL中设定了类名称)，将处理后的item存入MySQL中：

```
# 导入pymysql
import pymysql

class MysqlPipeline():
    def __init__(self, host, database, user, password, port):
        self.host = host
        self.database = database
        self.user = user
        self.password = password
        self.port = port
    
    # @classmethod 标识，from_crawler是类方法，用来获取settings.py中的配置，参数是 crawler，通过 crawler拿到配置信息，进行返回
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            host=crawler.settings.get('MYSQL_HOST'),
            database=crawler.settings.get('MYSQL_DATABASE'),
            user=crawler.settings.get('MYSQL_USER'),
            password=crawler.settings.get('MYSQL_PASSWORD'),
            port=crawler.settings.get('MYSQL_PORT'),
        )

    def open_spider(self, spider):
        self.db = pymysql.connect(self.host, self.user, self.password, self.database, charset='utf8', port=self.port)
        self.cursor = self.db.cursor()

    def close_spider(self, spider):
        self.db.close()

    # 最主要的process_item（）方法则执行了数据插入操作
    def process_item(self, item, spider):
        print(dict(item))
        data = dict(item)
        keys = ', '.join(data.keys())
        values = ', '.join(['%s'] * len(data))
        sql = 'insert into %s (%s) values (%s)' % (item.table, keys, values)
        self.cursor.execute(sql, tuple(data.values()))
        self.db.commit()
        return item
```

##### 保存MongoDB数据库

在pipelines.py 文件添加一个类MongoPipeline(在上面的设置MongoDB中设定了类名称)，将处理后的item存入MongoDB中：

```
# 导入pymongo
import pymongo

class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    # @classmethod 标识，from_crawler是类方法，用来获取settings.py中的配置，参数是 crawler，通过 crawler拿到配置信息，进行返回
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        	)

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    # 最主要的process_item（）方法则执行了数据插入操作
    def process_item(self, item, spider):
    	# 获取数据表名
        name = item.table
        self.db[name].insert(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

### Items文件

**Item 是保存爬取数据的容器，定义需要爬取的字段。创建 Item 需要继承 scrapy.Item 类，并且定义类型为 scrapy.Field 字段。**

这里创建了一个新的类QuoteItem，和三个新的字段text、author、tags。

```
import scrapy

class QuoteItem(scrapy.Item):
    # 设置数据表名
    table = '...'
    # 设置字段名
    text = scrapy.Field()
    author = scrapy.Field()
    tags = scrapy.Field()
```

### 创建Spider文件

**注意：创建spider时，要先进入第一层test1项目文件夹**

```
# 进入test1项目文件夹
cd test1

# 创建一个名为chong的爬虫，爬取的url设定为quotes.toscrape.com
scrapy genspider chong quotes.toscrape.com
```

**这时在spiders文件夹里面多出一个名为chong.py的爬虫文件**，打开文件出现以下内容：

```
import scrapy

class ChongSpider(scrapy.Spider):

    # name定义了这个爬虫的名为chong，name用来区分不同的爬虫。
    name = 'chong'
    
    # allowed_domains定义了允许爬取的域名，若请求链接不是这个域名下的，会被过滤掉
    allowed_domains = ['quotes.toscrape.com']
    
    # start_urls定义了初始的请求
    start_urls = ['http://quotes.toscrape.com/']

    # parse方法被调用时，上面start_urls请求就完成了，并返回response作为唯一参数传递给这个函数。
    def parse(self, response):
        pass
```

##### 选择器

Scrapy 框架自带选择器，即 Selector，基于lxml构建，支持Xpath、CSS 选择器及正则表达式，解析速度和准确度非常高。

Selector 是一个可以独立使用的模块，利用 Selector 这个类来构建一个选择器对象，然后调用它的相关方法 Xpath、CSS选择器等来提取数据。

```
# 导入Selector
from scrapy import Selector

body='<html><head><title>Hello World</title></head><body></body></html> '
# 构建Selector选择器对象，传入text参数。
selector = Selector(text=body)
# 调用xpath()方法来提取
title = selector.xpath('//title/text()').extract_first()
print(title)

结果：
Hello World
```

在items中定义了新字段，在parse()方法中添加提取字段的方法，方法可以是CSS或Xpath选择器，这里用CSS选择器：

```
# 从items.py文件导入QuoteItem类实例化（注意：导入的时候要删除最前面的项目文件夹名称及 . ）
from test1.items import QuoteItem

import scrapy

class ChongSpider(scrapy.Spider):
    name = 'chong'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    # response是上面start_urls请求返回的响应，直接拿来用。
    def parse(self, response):
        # 这里选用CSS选择器，选择class为quote的元素，赋值给quotes
        quotes = response.css('.quote')
        # item可以认为一个字典，解析字段内容，赋值成了一个个QuoteItem
        item = QuoteItem()
        # 遍历quotes
        for quote in quotes:
            # 选取class为text的元素，::text获取文本内容，extract_first()获取第一个元素
            item['text'] = quote.css('.text::text').extract_first()
            # 选取class为author的第一个元素文本
            item['author'] = quote.css('.author::text').extract_first()
            # 选取class为tags下面的class为tag的元素所有文本，extract()获取整个列表
            item['tags'] = quote.css('.tags .tag::text').extract() 
	    # 返回item
	    yield item
	
	# 循环爬取
	# 获取下一个页面链接，::attr(href)超链接a中href属性，extract_first()获取第一个元素内容
        next = response.css('.pager .next a::attr(href)').extract_first()
        
	# urljoin()构造成绝对url，例如：获取到的下一页地址是/page/2，urljoin()处理后就是http://quotes.toscrape.com/page/2/
        url = response.urljoin(next)
        
	# 构造请求
	# 构造请求时需要用到scrapy.Request，传递两个参数url和callback，url：请求链接，callback：回调函数
        # 当前请求完成后，响应会重新经过parse方法，得到第二页的解析结果，往复迭代，直到最后一页。
        yield scrapy.Request(url=url, callback=self.parse)
```

有时候在爬取的时候出现错误，DEBUG：Filtered offsite request to 域名，这是因为请求的url域名和上面的allowed_domains不一致，被过滤掉。

```
# 解决办法可以停用过滤功能，如下所示：

yield scrapy.Request(url,callback=self.parse,dont_filter=True)
```

### 运行爬虫

**注意：运行爬虫时，位置要在项目文件夹内，要不然Scrapy找不到命令集。**

运行爬虫的命令：

```
scrapy crawl chong（爬虫的名称）
```

```
部分结果：
{'downloader/request_bytes': 2903,
 'downloader/request_count': 11,
 'downloader/request_method_count/GET': 11,
 'downloader/response_bytes': 24812,
 'downloader/response_count': 11,
 'downloader/response_status_count/200': 10,
 'downloader/response_status_count/404': 1,
 'dupefilter/filtered': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2018, 11, 28, 15, 50, 36, 962576),
 'item_scraped_count': 100,
 'log_count/DEBUG': 112,
 'log_count/INFO': 7,
 'request_depth_max': 10,
 'response_received_count': 11,
 'scheduler/dequeued': 10,
 'scheduler/dequeued/memory': 10,
 'scheduler/enqueued': 10,
 'scheduler/enqueued/memory': 10,
 'start_time': datetime.datetime(2018, 11, 28, 15, 50, 30, 765212)}
2018-11-28 23:50:36 [scrapy.core.engine] INFO: Spider closed (finished)
```

最后， Scrapy 输出了整个抓取过程的统计信息，如请求的字节数、请求次数、响应次数等。整个 Scrapy 程序成功运行，通过非常简单的代码就完成了一个网站内容的爬取。

##### 生成数据文件

保存数据命令：	

```
scrapy crawl chong -o chong.json
```

运行命令后，项目内多了一个chong.json文件，文件包含了刚才抓取的所有内容，内容格式是JSON格式。支持输出的格式还有许多种，例csv、xml、pickle等，输出时只需要将命令的后缀改为需要的格式名称即可。

# Scrapy方法

### 配置分类

##### 全局配置

在Scrapy框架中，全局配置指的是settings.py文件中定义的配置，配置后即生效。

##### 局部配置

在Scrapy框架中，局部配置指的是middlewares.py、pipeline.py文件中定义的配置，**配置后需要在settings.py文件中添加并取消注释相应的中间件才生效**。

**注意：在Scrapy框架中，局部配置优先于全局配置，且运行框架时，全局配置只读取一次，局部配置是一直被读取。**

### 设置代理IP

##### 功能包添加代理IP

当我们在爬取网站时，就只有当前一个IP地址，若被设置反爬的服务器封掉IP，再去访问页面就会反馈给我们403（禁止访问）的页面。

**ProxyPool-master文件夹**

ProxyPool-master是一个通过免费代理网站，访问设定的网址，晒选出成功访问的代理ip的功能包。

我们将包装有代理ip功能的ProxyPool-master文件夹放到与上面项目文件夹相邻的地方，方便项目管理。

**设置ProxyPool-master参数**

在ProxyPool-master文件夹里面的setting文件设置参数：

TEST_URL后面设置代理ip要测试访问的网址

```
TEST_URL = 'https://www.....com'
```

API_PORT设置端口

```
API_PORT = 5555
```

**启动ProxyPool-master**

设置成功后，启动ProxyPool-master文件夹里面的run文件，启动成功后，它就会自动筛选有效代理IP，将IP存入代理IP池中。

当我们访问http://127.0.0.1:5555/count 这个接口时，它就会返回当前代理池中可用的代理IP数量。

当我们访问http://127.0.0.1:5555/random 这个接口时，它就会随机反馈一个代理池中的IP及端口号。

当我们启动上面的代理池后，我们需要在项目里面从接口获取代理IP

**middlewares文件**

在项目文件的middlewares.py文件，新建ProxyMiddleware类，添加代理ip:

```
# 导入requests库
import requests

# 添加代理IP中间件
class ProxyMiddleware(object):

    def process_request(self, request, spider):
    	  # 访问代理池提供的接口，随机获取代理IP
        pro_addr = requests.get('http://127.0.0.1:5555/random').text
        request.meta['proxy'] = 'http://' + pro_addr
```

**settings文件**

在settings文件取消DOWNLOADER_MIDDLEWARES注释，并添加上面代理IP的类方法

```
DOWNLOADER_MIDDLEWARES = {
    # zhipin项目名称，middlewares文件名称，ProxyMiddleware类名称
    'zhipin.middlewares.ProxyMiddleware': 543,
}
```

##### 中间件设置代理IP

**middlewares.py文件**

```
import scrapy
from scrapy import signals
import random

class ProxyMiddleware(object):
    '''
    设置Proxy
    '''

    def __init__(self, ip):
        self.ip = ip

    @classmethod
    def from_crawler(cls, crawler):
        return cls(ip=crawler.settings.get('PROXIES'))

    def process_request(self, request, spider):
        ip = random.choice(self.ip)
        request.meta['proxy'] = ip
```

**settings文件**

```
# 代理IP
PROXIES = ['http://183.207.95.27:80', 'http://111.6.100.99:80', 'http://122.72.99.103:80'...]

# 取消DOWNLOADER_MIDDLEWARES注释，并添加下面代理IP的类方法
DOWNLOADER_MIDDLEWARES = {
    # zhipin项目名称，middlewares文件名称，ProxyMiddleware类名称
    'zhipin.middlewares.ProxyMiddleware': 543,
}
```

### 设置请求头

在前面项目中，虽然settings.py文件中设置了随机的请求头，但这样设置有一个缺点就是：**爬虫爬取过程中不会更换请求头，这是因为在框架启动后，只会读取一次settings.py文件，只有重启框架，爬虫才会更换一次请求头。要想在爬虫爬取的过程中更换请求头，就需要添加请求头中间件。**

##### 中间件设置请求头

**middlewares.py文件**

```
from fake_useragent import UserAgent

class UserAgentMiddleware(object):
    def __init__ (self):
        self.ua = UserAgent().random

    def process_request(self,request,spider):
        request.headers['Use-Agent'] = self.ua
```

**settings文件**

在settings文件取消DOWNLOADER_MIDDLEWARES注释，并添加上面中间件

```
DOWNLOADER_MIDDLEWARES = {
    # zhipin项目名称，middlewares文件名称，UserAgentMiddleware类名称
    'zhipin.middlewares.UserAgentMiddleware': 543,
}
```

### 设置保存数据

有时候我们需要对爬取的数据进行编辑后，再保存，就需要在pipelines.py文件定义新的类方法。

##### 筛选长度

**pipelines.py文件**

```
from scrapy.exceptions import DropItem

# 定义一个类，筛掉text长度大于50的item
class Test1Pipeline(object):
    def __init__(self):
    	# 设置长度为50
        self.limit = 50

    def process_item(self, item, spider):
        if item['text']:
            if len(item['text']) > self.limit:
            	# 如果长度大于50，切片，50长度后面的用...代替
                item['text'] = item['text'][0:self.limit].rstrip()+'...'
            return item
        else:
            return DropItem('Missing Text')
```

**settings.py文件**

在settings.py文件，取消ITEM_PIPELINES(项目管道)注释，并添加上面类方法。

**注意：这里是将爬取的数据进行筛选，因此是添加在ITEM_PIPELINES当中的。**

**注意：这里的添加筛选长度的类方法后面的数值，必须比存储数据的类方法后面的值要小，因为越小，越先执行。**

```
 ITEM_PIPELINES = {
   # zhipin项目名称，pipelines文件名称，Test1Pipeline类名称
   'zhipin.pipelines.Test1Pipeline': 300,
}
```

##### 去重

一个用于去重的过滤器，丢弃那些已经被处理过的item。让我们假设我们的item有一个唯一的id，但是我们spider返回的多个item中包含有相同的id:

**pipelines.py文件**

```
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):

    def __init__(self):
        # 新建一个空集合
        self.ids_seen = set()

    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
```

**settings.py文件**

```
ITEM_PIPELINES = {
   # zhipin项目名称，pipelines文件名称，DuplicatesPipeline类名称
   'zhipin.pipelines.DuplicatesPipeline': 300,
}
```

##### 过滤

同过上面的使用方法，我们也可以写一个过滤器，筛选掉无效数据。

**pipelines.py文件**

```
from scrapy.exceptions import DropItem

class ScreenPipeline(object):

    def __init__(self):
        pass

    def process_item(self, item, spider):
        if item['name']:
            return item
        else:
            raise DropItem("无效数据")
```

**settings.py文件**

```
ITEM_PIPELINES = {
   # zhipin项目名称，pipelines文件名称，ScreenPipeline类名称
   'zhipin.pipelines.ScreenPipeline': 300,
}
```

### 设置新response

在初始定义的spider文件中，只有一个parse方法来爬取网页信息，而第一次使用的parse方法中的response就是访问初始请求start_urls中的地址返回的响应。

**spider文件**

```
import scrapy

class ChongSpider(scrapy.Spider):

    # name定义了这个爬虫的名为chong，name用来区分不同的爬虫。
    name = 'chong'
    
    # allowed_domains定义了允许爬取的域名，若请求链接不是这个域名下的，会被过滤掉
    allowed_domains = ['quotes.toscrape.com']
    
    # start_urls定义了初始的请求
    start_urls = ['http://quotes.toscrape.com/']

    # parse方法被调用时，上面start_urls请求就完成了，并返回response作为唯一参数传递给这个函数。
    def parse(self, response):
        pass
```

有时候，我们需要使用的response不是初始请求start_url返回的响应，就需要新增一个start_requests方法来重新定义响应。

**注意：设置新response的方法的名称只能是start_requests，不能随意命名。**

```
import scrapy

class ChongSpider(scrapy.Spider):

    # name定义了这个爬虫的名为chong，name用来区分不同的爬虫。
    name = 'chong'
    
    # allowed_domains定义了允许爬取的域名，若请求链接不是这个域名下的，会被过滤掉
    allowed_domains = ['quotes.toscrape.com']
    
    # start_urls定义了初始的请求
    start_urls = ['http://quotes.toscrape.com/']
    
    # 新增start_requests方法
    def start_requests(self):
        # 定义新的url
        url = f'https://www.baidu.com'
        # 调用parse方法
        yield scrapy.Request(url=url, callback=self.parse)

    # parse方法被调用时，上面start_requests的url请求就完成了，response就是这个url的响应，就不再是start_urls的响应。
    def parse(self, response):
        pass
```

### 断点续爬

在爬虫运行过程中，可能由于网络问题、或者突然断电等多种原因，导致爬虫意外结束。如果重启爬虫，前面爬取的数据又需要重新爬取一次，浪费时间；若舍弃没有爬取的数据，就会造成数据的不完整。因此最好的办法就是将已经爬取的url保存起来，实现断点续爬。

这里我们使用Redis数据库的集合类型set来保存url

**spier文件**

```
import redis
import scrapy

class ChongSpider(scrapy.Spider):
    # 连接Redis
    con = redis.StrictRedis(host="127.0.0.1",port=6379)

    def start_requests(self):
        pass

    def parse(self, response):
        # 将已下载页面的url存入Redis的save_url中
        self.con.sadd('save_url',response.request.url)
        ...
        # 如果新构造的url不存在在Redis的save_url中，再执行parse方法
        if not self.con.sismember('save_url', url):
            yield scrapy.Request(url=url, callback=self.parse, dont_filter=True)
```

# Scrapy实战

##### 目标：

​	爬取 360 摄影美图，通过Ajax来获取信息，来分别实现 Mongo DB 存储、MySQL 存储、Image 图片存储的三个Pipeline。

### images360项目

##### 开始步骤

新建一个名为images360的项目

```
scrapy startproject images360
```

修改 settings.py 中的 ROBOTSTXT_OBEY 变量，将其设置为 False ，否则无法抓取。

```
ROBOTSTXT_OBEY = False
```

进入images360项目文件夹，创建一个名为images，爬取的url为images.so.com的Spider

```
cd images360
scrapy genspider images images.so.com
```

这样就建立了基本的框架，接下来就是要修改、添加项目代码。

##### 添加代码

​	在settings.py文件中，定义一个爬取网页数目的变量MAX_PAGE：

```
MAX_PAGE = 50
```

​	在images.py爬虫文件中，定义一个start_requests（）方法来生成与网页数目一样次数的请求：

```
from scrapy import Request
from urllib.parse import urlencode

    def start_requests(self):
    	
    	# 将url不变的参数做成data字典，方便形成新的url
        data = {'ch': 'photography', 'listtype': 'new'}
        
        # 将url前面部分赋值为base_url变量
        base_url = 'https://image.so.com/zj?'
       
       	# 生成与网页数目一样次数的请求
        for page in range(1, self.settings.get('MAX_PAGE') + 1):
        
        	# 给url添加可变参数，规律为page页码的30倍
            data['sn'] = page * 30
            
            # 将data字典通过urlencode转换成url的GET参数
            # urlencode({'ch': '1', 'en': '0'}) == ch=1&en=0
            params = urlencode(data)
            
            # 形成完整的url
            url = base_url + params
            
            # 将url返回给parse()方法
            yield Request(url, self.parse)
            
# 注意：将url返回给parse()时，会访问url，自动生成reponse，直接在parse()用。
```

##### 定义Item

定义一个item，叫做ImageItem。

```
class ImageItem(scrapy.Item):

    collection = table = 'images'
    id = scrapy.Field()
    url = scrapy.Field()
    title = scrapy.Field()
    thumb = scrapy.Field()
```

​	这里将两个属性collection和table定义为images字符串，分别代表MongoDB的存储的Collection名称和MySQL存储的表名称，还定义了4个字段。

##### 改写Spider里面的parse()方法

```
import json
# 导入ImageItem类
from images360.items import ImageItem

    def parse(self, response):
        result = json.loads(response.text)
        for image in result.get('list'):
            item = ImageItem()
            item['id'] = image.get('imageid')
            item['url'] = image.get('qhimg_url')
            item['title'] = image.get('group_title')
            item['thumb'] = image.get('qhimg_thumb_url')
            yield item
```

​	首先解析JSON，遍历器list字段，取出一个个图片信息，然后再对ImageItem赋值，生成Item对象。

##### 将信息保存到MongoDB和MySQL

##### 1.MongoDB设置

​	在settings.py文件里添加变量：

```
# 连接地址
MONGO_URI = '127.0.0.1'
# 连接的数据库
MONGO_DB = 'images360'

# 设置ITEM_PIPELINES中添加'images360.pipelines.MongoPipeline': 301
```

​	在pipeline.py文件添加代码：

```
import pymongo

class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def process_item(self, item, spider):
    	# 设置数据表名
        name = item.collection
        self.db[name].insert(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close() 

```

##### 2.MySQL设置

​	MySQL不同于MongoDB，MongoDB数据库不存在，使用时会自动新建，MySQL则要手动先建立数据库再使用。

​	新建一个名为images360数据库：

```
create database images360 default character set utf8 collate utf8_general_ci;
```

​	新建一个名为images数据表，包含id、url、title、thumb四个字段

```
use images360;
# 新建数据表
create table images(id varchar(255) primary key, url varchar(255) null, title varchar(255) null, thumb varchar(255) null);
```

​	在settings.py文件里添加变量：

```
MYSQL_HOST = '127.0.0.1'
MYSQL_DATABASE = 'images360'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '5335915'
MYSQL_PORT = 3306

# 设置ITEM_PIPELINES中添加'images360.pipelines.MysqlPipeline': 302,
```

​	在pipeline.py文件添加代码：

```
import pymysql

class MysqlPipeline():
    def __init__(self, host, database, user, password, port):
        self.host = host
        self.database = database
        self.user = user
        self.password = password
        self.port = port
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            host=crawler.settings.get('MYSQL_HOST'),
            database=crawler.settings.get('MYSQL_DATABASE'),
            user=crawler.settings.get('MYSQL_USER'),
            password=crawler.settings.get('MYSQL_PASSWORD'),
            port=crawler.settings.get('MYSQL_PORT'),
        )
    
    def open_spider(self, spider):
        self.db = pymysql.connect(self.host, self.user, self.password, self.database, charset='utf8',
                                  port=self.port)
        self.cursor = self.db.cursor()
    
    def close_spider(self, spider):
        self.db.close()
    
    def process_item(self, item, spider):
        data = dict(item)
        keys = ', '.join(data.keys())
        values = ', '.join(['%s'] * len(data))
        sql = 'insert into %s (%s) values (%s)' % ('表名', keys, values)
        self.cursor.execute(sql, tuple(data.values()))
        self.db.commit()
        return item
```

##### Image Pipeline

​	Scrapy提供了专门处理下载的Pipeline，包括文件下载和图片下载。下载文件和图片的原理与抓取页面原理一样，下载过程支持异步和多线程，十分高效。

​	在settings.py里面定义一个变量IMAGES_STORE，设置存储文件路径：

```
IMAGES_STORE = './images'

# 设置ITEM_PIPELINES中添加'images360.pipelines.ImagePipeline': 300,
```

​	将路径定义为当前路径下的images子文件夹，即下载的图片都会保存到本项目的images文件夹中。

​	在pipeline.py文件定义ImagePipeline：

```
from scrapy import Request
from scrapy.exceptions import DropItem
from scrapy.pipelines.images import ImagesPipeline

class ImagePipeline(ImagesPipeline):
    
    # file_path()它的第一个参数request就是当前下载对应的Request对象。
    def file_path(self, request, response=None, info=None):
        url = request.url
        file_name = url.split('/')[-1]
        return file_name
    
    # item_completed()，它是当单个 Item 完成下载时的处理方法。
    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem('Image Downloaded Failed')
        return item
    
    # get_media_requests()它的第一个参数 item 是爬取生成的 Item对象。
    def get_media_requests(self, item, info):
        yield Request(item['url'])
```

##### 运行spider

```
scrapy crawl images
```

​	注意：在settings.py中要优先调用ImagePipeline对Item做下载筛选，下载失败的 Item
直接忽略，它们就不会保存到 MongoDB和MySQL中 。随后再调用其他两个存储的 Pipeline ，这样
确保存入数据库的图片都是下载成功的。

