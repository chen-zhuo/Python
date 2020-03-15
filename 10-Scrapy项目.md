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
