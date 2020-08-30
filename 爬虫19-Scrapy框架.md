# Scrapy框架

Scrapy文档参看：[Scrapy官方文档](https://doc.scrapy.org/en/latest/intro/overview.html#)  

### Scrapy简介

**Scrapy 是一个基于 Twisted 的异步处理纯 Python 实现的爬虫框架。**架构清晰， 模块之间的耦合程度低（相互影响小），可扩展性极强，可以灵活完成各种需求，我们只需要定制开发几个模块就可以轻松实现一个爬虫。

在任何情况下，都不要写阻塞的代码。例如：

1. 访问文件、数据库或者Web

2. 产生新的进程并需要处理新进程的输出，如运行shell命令

3. 执行系统层次操作的代码，如等待系统队列

##### Scrapy安装

Scrapy是一个完整的爬虫框架，安装过程中会涉及到许多依赖库，需要一步一步安装：

**pip安装**

因为安装Scrapy框架需要许多的依赖库，按照下面步骤通过 `pip` 命令一步步安装：

?> 有的库是直接程序就可以安装，有的库需要下载下来手动安装。手动安装的库，需要在网站内选择适合自己环境的版本下载安装：参看[基础02-开发工具、包管理命令、镜像源](基础02-开发工具、包管理命令、镜像源.md)。

1. 安装wheel库：命令行中输入`pip install wheel`命令进行安装。
2. 安装lxml库：在 [http://www.lfd.uci.edu/~gohlke/pythonlibs/](http://www.lfd.uci.edu/~gohlke/pythonlibs/) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
3. 安装zope.interface库：在 [https://pypi.python.org/pypi/zope.interface#downloads](https://pypi.python.org/pypi/zope.interface#downloads) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
4. 安装pyOpenssl库：在 [https: //pypi python.org pypi pyOpenSSL#downloads](https: //pypi python.org pypi pyOpenSSL#downloads) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
5. 安装Twisted库：在 [http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted](http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。
6. 安装pywin32库：在 [https://sourceforge.net/projects/pywin32/files/pywin32/Build%20220/](https://sourceforge.net/projects/pywin32/files/pywin32/Build 220/) 找到适合的版本，采用pip方式安装`pip install 路径/下载的文件.whl`。但是容易出现问题，就是在安装时会出现提示python3.6-32在注册表中不存在。首先根据 [pywin32不能安装解决方案](http://blog.csdn.net/u014680513/article/details/51005650) 把Python写入注册表，在HKEY-CURRENT_USER–Software–Python–PythonCore–会发现3.6文件夹，把这个文件夹导出并重命名为3.6-32，然后导入注册表。就可以安装Pywin32了。
7. 安装Scrapy框架：命令行中输入`pip install scrapy`命令进行安装。

**conda安装**

因为conda会解决依赖包的问题，所以安装命令也很简单：

```
conda install scrapy
```

?> 在安装Scrapy时，conda会收集安装Scrapy所需要的依赖包，安装好后，再安装scrapy框架。

##### Scrapy架构

![1370011-a3be891e6a48dad7](image/1370011-a3be891e6a48dad7.webp)

```
Engine:引擎，处理整个系统的数据流处理、触发事务，是整个框架的核心。
Item:项目，它定义了爬取结果的数据结构，爬取的数据会被赋值成该 Item 对象。
Scheduler:调度器，接受引擎发过来的请求并将其列中，在引擎再次请求的时候将请求提供给引擎。
Downloader:下载器，下载网页内容，并将网页容返回给蜘蛛。
Spiders:蜘蛛，其内定义了爬取的逻辑和网页 解析规 ，它主要负责解析响应并生成提取结果和新的请求。
ItemPipelne:项目管道，负责处理由蜘蛛从网页中取的项目，它的主要任务是清洗、验证和存储数据。
Downloader Middlewares:下载器中间件，位于引擎和下载器之间的钩子框架，主要处理引擎与下载器之间的请求及响应。
Spide Middlewares:蜘蛛中间件，位于引擎和蜘蛛之间的钩子框架，主要处理蜘蛛输入的响应和输出的结果及新的请求。
```

##### Scrapy数据流

```
1、引擎打开起点网站(open a domain)，向处理该网站的Spider请求第一个要爬取的URL。
2、引擎从Spider中获取到第⼀个要爬取的URL并在调度器(Scheduler)调度。
3、引擎向调度器请求下⼀个要爬取的URL。
4、调度器返回下⼀个要爬取的URL给引擎，引擎将URL通过下载中间件转发给下载器(Downloader)。
5、页面下载完毕，下载器生成页面的Response，并将其通过下载中间件发送给引擎。
6、引擎从下载器中接收到Response并通过Spider中间件发送给Spider处理。
7、Spider在Response中爬取到的Item和新的Request给引擎。
8、引擎将爬取到的Item给Item Pipeline，将Request给调度器。
9、重复第二步到第八步直到调度器中没有更多地request，引擎关闭该网站。
```

### Scrapy项目

##### 创建项目文件

命令行选择项目创建的所在路径，执行下面命令就会在路径下面生成名称为`test1` 的scrapy项目：

```
scrapy startproject test1（项目名称）
```

成功创建示例：

```
(base) C:\Users>scrapy startproject test1
New Scrapy project 'test1', using template directory 'D:\Program Files\anaconda3\lib\site-packages\scrapy\templates\project', created in :
    C:\Users\test1

You can start your first spider with:
    cd test1
    scrapy genspider example example.com
```

展开test1项目文件夹下面的所有文件夹，可以看到其全部的项目结构：

![QQ截图20200805235817](image/QQ截图20200805235817.png)

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

文件信息：

```
spiders文件夹：专门存放Spiders文件的文件夹。
items.py：它定义 Item 数据结构。
middlewares.py：它定义 Spider Middlewares 和 Downloader Middlewarese的实现。
pipelines.py：它定义 Item Pipeline 的实现。
settings.py：它定义项目的全局配置。
scrapy.cfg：它是 Scrapy 项目的配置文件，其内定义了项目的配置文件路径、部署相关信息等内容。
```

##### 创建爬虫文件

创建的爬虫文件**自动存放在上面提到了 `test1` 项目文件夹里面的 `spiders` 文件夹里面**。

```
# 进入test1项目文件夹
cd test1

# 创建一个名称为quotes.py的爬虫文件，采集的网络域名为quotes.toscrape.com
scrapy genspider quotes quotes.toscrape.com
```

成功创建示例：

```
(base) C:\Users\ChenZhuo\Desktop\Python Example>cd test1

(base) C:\Users\ChenZhuo\Desktop\Python Example\test1>scrapy genspider quotes quotes.toscrape.com
Created spider 'quotes' using template 'basic' in module:
  test1.spiders.quotes
```

创建成功后，会在 `spiders` 文件夹里面多出一个 `quotes.py`的爬虫文件：

```
test1
	-test1
		-spiders
			__init__.py
			quotes.py
		__init__.py
		items.py
		middlewares.py
		pipelines.py
		settings.py
	scrapy.cfg	
```

![QQ截图20200806001613](image/QQ截图20200806001613.png)

```python
import scrapy

# 从网页里抓取内容，并解析抓取的结果，不过这个类必须继承Scrapy提供的Spider类，即scrapy.Spider
class QuotesSpider(scrapy.Spider):
    # 爬虫的唯一名称，用来区分不同的爬虫
    name = 'quotes'
    # 允许爬取的域名，如果初始或后续的请求链接不是这个域名下的，则请求链接会被过滤掉
    allowed_domains = ['quotes.toscrape.com']
    # spider启动时爬取的url列表，初始请求是由它来定义的
    start_urls = ['http://quotes.toscrape.com/']
    
    # 负责解析start_urls中链接返回的response响应、提取数据或者进一步生成要处理的请求
    def parse(self, response):
        pass
```

##### Item数据结构

items.py文件作用是定义Item数据结构。

**Item是保存爬取数据的容器，使用方法和字典类似。创建Item需要继承 `scrapy.Item` 类，并定义类型为 `scrapy.Field` 的字段。**

![QQ截图20200809165708](image/QQ截图20200809165708.png)

```python
import scrapy

# 观察网站，我们定义三个我们需要采集的字段
class Test1Item(scrapy.Item):
    # 文本内容
    text = scrapy.Field()
    # 作者
    author = scrapy.Field()
    # 标签
    tags = scrapy.Field()
```

##### 解析Response

定义好了items.py中的字段，回到爬虫文件解析Response响应中的内容，赋值给定义好的字段：

![QQ截图20200812231308](image/QQ截图20200812231308.png)

```python
import scrapy
# 导入items.py中的Test1Item类
from ..items import Test1Item


class QuotesSpider(scrapy.Spider):
    # name定义了这个爬虫的名称为quotes，作用是用来区分不同的爬虫
    name = 'quotes'
    # allowed_domains定义了允许爬取的域名，若请求链接不是这个域名下的，会被过滤掉
    allowed_domains = ['quotes.toscrape.com']
    # start_urls定义了初始的请求
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # 这里用CSS选择器来提取内容
        quotes = response.css('.quote')
        # 根据网页代码标签来遍历
        for quote in quotes:
            # 实例化Item
            item = Test1Item()
            # Item使用方法和字典类似
            # 给Item结构中的text文本内容赋值
            item['text'] = quote.css('.text::text').extract_first()
            # 给Item结构中的author作者赋值
            item['author'] = quote.css('.author::text').extract_first()
            # 给Item结构中的tags标签赋值
            item['tags'] = quote.css('.tags .tag::text').extract()
            # 最后将Item返回
            yield item
```

##### 生成Resquest

上面解析Response只是解析了第一页的内容，**并没有生成后续的Resquest，也就无法采集后面页数，因此需要构造下一个Resquest，循环往复，全部采集。**

构造请求时需要用到 `scrapy.Request`，这里传递两个参数：

- `url`：下一个请求链接。
- `callback`：回调函数。当指定的回调函数的请求完成之后，获取到响应，引擎会将该响应作为参数传递给这个回调函数，进行解析或生成下一个请求。

![QQ截图20200812231546](image/QQ截图20200812231546.png)

```python
        # 获取下一页的url(例如：/page/2)
        next = response.css('.pager .next a::attr("href")').extract_first()
        # 将相对url构造为绝对url(例如：http://quotes.toscrape.com/page/2)
        url = response.urljoin(next)
        # 因为下一页的结构和当前解析页类似，仍然使用parse方法解析
        yield scrapy.Request(url=url, callback=self.parse)
```

**重点**：当爬取到最后一页时，下一页的url值为 `None`， 即 `response.urljoin(next)` 为 `response.urljoin(None)` ，但它的值等于上一次 `response.urljoin(next)` 的值，这样就会导致最后的一页的url出现两次，从而终止循环。

**过滤功能**：有时候在爬取的时候出现错误 `DEBUG:Filtered offsite request to 域名`，这是因为请求的url域名和上面的 `allowed_domains` 不一致，被过滤掉。

```python
# 解决办法可以停用过滤功能，如下所示：
yield scrapy.Request(url=url, callback=self.parse, dont_filter=True)
```

##### 运行爬虫文件

接下来，在命令行爬虫项目所在的路径下，输入下面命令，即可运行爬虫：

```python
# quotes就是爬虫的名字，即爬虫文件里面name变量
scrapy crawl quotes
```

![QQ截图20200816005006](image/QQ截图20200816005006.png)

##### 保存结果(文件)

如果想将上面的结果保存成JSON文件，可以执行下面命令：

```
scrapy crawl quotes -o quotes.json
```

输出格式还支持很多种，例如 `csv、xml、pickle、marsha` 等，还支持 `ftp、s3` 等远程输出：

```
scrapy crawl quotes -o quotes.csv 
scrapy crawl quotes -o quotes.xml 
scrapy crawl quotes -o quotes.pickle 
scrapy crawl quotes -o quotes.marshal 
scrapy crawl quotes -o ftp://user:pass@ftp.example.com/path/to/quotes.csv
```

##### 保存结果(数据库)

在 settings.py 的最下面设置在 `Item Pileline` 中使用到的数据库的相关信息：

```python
# 设置MySQL数据库
MYSQL_HOST = '127.0.0.1'
MYSQL_DATABASE = '...'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '...'
MYSQL_PORT = 3306

# 设置MongoDB数据库
MONGO_URI = '127.0.0.1'
MONGO_DB = '...'
```

**如果想将数据保存在数据库中，则可以在 pipelines.py 定义 `Item Pileline` 来实现，定义一个类并实现 `process_item()` 方法即可。启用 `Item Pipeline` 后会自动调用这个方法。`process_item()` 方法必须返回包含数据的字典或 Item 对象，或者抛出 `Dropltem` 异常。**

**保存到MySQL**：将处理后的Item存入MySQL数据库，同样在 pipelines.py 定义一个 `MysqlPipeline` 实现：

```python
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
    
    # 当Spider开启时，这个方法被调用，主要进行了一些初始化操作
    def open_spider(self, spider):
        self.db = pymysql.connect(self.host, self.user, self.password, self.database, charset='utf8', port=self.port)
        self.cursor = self.db.cursor()
     
    # 最主要的process_item()方法则执行了数据插入操作
    # 参数item：每次Spider生成的Item都会作为参数传递过来
    # 参数spider：就是Spider实例
    def process_item(self, item, spider):
        # 定义数据表名
        teble = 'Test1Item'
        data = dict(item)
        keys = ', '.join(data.keys())
        values = ', '.join(['%s'] * len(data))
        sql = 'insert into %s (%s) values (%s)' % (table, keys, values)
        self.cursor.execute(sql, tuple(data.values()))
        self.db.commit()
        # 返回Item对象
        return item
        
    # 当Spider关闭时，这个方法会调用，将数据库连接关闭
    def close_spider(self, spider):
        self.db.close()
```

**保存到MongoDB**：将处理后的Item存入MongoDB数据库，同样在 pipelines.py 定义一个 `MongoPipeline` 实现：

```python
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

    # 最主要的process_item()方法则执行了数据插入操作
    # 参数item：每次Spider生成的Item都会作为参数传递过来
    # 参数spider：就是Spider实例
    def process_item(self, item, spider):
    	# 定义数据表名
        name = 'Test1Item'
        self.db[name].insert(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

上面写了两个 `Item Pileline`，保存到MySQL `MysqlPipeline`、保存到MongoDB `MongoPipeline`，接下来就要在 settings.py 启用这些配置，启用 `Item Pileline`：

```Python
# 取消ITEM_PIPELINES注释
# test1项目名称，pipelines文件名称，...Pipeline在pipelines.py定义的类
ITEM_PIPELINES = {
   'test1.pipelines.MysqlPipeline': 400,
   'test1.pipelines.MongoPipeline': 500,
}
# 后面对应的键值代表调用优先级，数字越小越先被调用。也就是先存入MySQL，然后再存入MongoDB。
```

### Scrapy进阶

##### 禁用Robots协议

**Robots协议**：Robots协议（也称为爬虫协议、机器人协议等）的全称是“网络爬虫排除标准”（Robots Exclusion Protocol），网站通过Robots协议告诉爬虫和搜索引擎哪些页面可以抓取，哪些页面不能抓取。

在 settings.py 中，将 `ROBOTSTXT_OBEY` 取消注释，设置为 `False`。

```python
# 防止被爬网站的robots.txt起作用，需要关闭Robot协议检查
ROBOTSTXT_OBEY = False
```

##### Selector自带选择器

Scrapy 框架还自带了 `Selector选择器`，基于lxml构建，支持Xpath、CSS 选择器及正则表达式，解析速度和准确度非常高。

**Selector 是一个可以独立使用的模块**，利用 Selector 这个类来构建一个选择器对象，然后调用它的相关方法 Xpath、CSS选择器等来提取数据。

```python
# 这里没有在Scrapy框架中运行，而是单独把Selector拿来使用
from scrapy import Selector

body='<html><head><title>Hello World</title></head><body></body></html> '
# 构建Selector选择器对象，传入text参数。
selector = Selector(text=body)
# 调用xpath()方法来提取
title = selector.xpath('//title/text()').extract_first()
print(title)
'''
输出：
Hello World
'''
```

##### Middleware中间件

**Downloader Middleware(下载中间件)：一个处于 Scrapy 的 Request 和 Response 之间的处理模块，其功能十分强大，修改 User-Agent、处理重定向、设置代理、失败重试、设置 Cookies 等功能都需要借助它来实现**。在整个架构中起作用的位置是以下两个：

1. 在 Scheduler 调度出队列的 Request 发送给 Doanloader 下载之前，也就是我们可以在 Request 执行下载前对其修改。
2. 在下载后生成 Response 发送给 Spider 之前，也就是我们可以在生成 Response 被 Spider 解析之前对其修改。

Downloader Middleware核心方法有三个，**只需要实现至少一个方法，就可以定义一个 Downloader Middleware**：

```python
# request：即Request对象，即被处理的Request
# spider：即Spider对象，即此Request对应的Spider
process_request(request, spider)
'''
Request被Scrapy引擎调度给Downloader之前，process_request()方法就会被调用，也就是在Request从队列里调度出来到Downloader下载执行之前，我们都可以用process_request()方法对Request进行处理。
'''

# request：即Request对象，即此Response对应的Request
# response：即Response对象，即此被处理的Response
# spider：即Spider对象，即此Response对应的Spider
process_response(request, response, spider)
'''
Downloader执行Request下载之后，会得到对应的Response。Scrapy引擎便会将Response发送给Spider进行解析。在发送之前，我们都可以用process_response()方法来对Response进行处理。
'''

# request：即Request对象，即产生异常的Request
# exception：即Exception对象，即抛出的异常
# spider：即Spider对象，即此Request对应的Spider
process_exception(request, exception, spider)
'''
当Downloader或process_request()方法抛出异常时，例如抛出IgnoreRequest异常，process_exception()方法就会被调用。
'''
```

**Spider Middleware(爬虫中间件)：介入到 Scrapy 的 Spider 处理机制的钩子框架。**有如下三个作用：

1. 在 Downloader 生成的 Response 发送给 Spider 之前，对 Response 进行处理。
2. 在 Spider 生成的 Request 发送给 Scheduler 之前，对 Request 进行处理。
3. 在 Spider 生成的 Item 发送给 Item Pipeline 之前，对 Item 进行处理。

Spider Middleware核心方法有四个，**只需要实现至少一个方法，就可以定义一个 Spider Middleware**：

```python
# response：即Response对象，即被处理的Response
# spider：即Spider对象，即该Response对应的Spider
process_spider_input(response, spider)
'''
当Response被Spider Middleware处理时，process_spider_input()方法被调用。
'''

# response：即Response对象，即生成该输出的Response
# result：包含Resquest或Item对象的可迭代对象，即Spider返回的结果
# spider：即Spider对象，即其结果对应的Spider
process_spider_output(response, result, spider)
'''
当Spider处理Response返回结果时，process_spider_output()方法被调用。
'''

# response：即Response对象，即异常被抛出时被处理的 Response
# exception：即Exception对象，即被抛出的异常
# spider：即Spider对象，即抛出该异常的Spider
process_spider_exception(response, exception, spider)
'''
当Spider或Spider Middleware的process_spider_input()方法抛出异常时，process_spider_exception()方法被调用。
'''

# start_requests：即包含Request的可迭代对象，即Start Requests。
# spider：即Spider对象，即Start Requests所属的Spider
process_start_requests(start_requests, spider)
'''
process_start_requests()方法以Spider启动Request参数被调用，执行的过程类似于process_spider_output()，只不过它没有相关联的Response，并且必须返回Request。
'''
```

##### 设置新response

在初始定义的spider文件中，只有一个parse方法来爬取网页信息，而第一次使用的parse方法中的response就是访问初始请求 `start_urls` 中的地址返回的响应。

```python
import scrapy

class ChongSpider(scrapy.Spider):
    name = 'chong'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    # parse方法被调用时，上面start_urls请求就完成了，并返回response作为唯一参数传递给这个函数。
    def parse(self, response):
        pass
```

但如果我们需要使用的response不是初始请求start_url返回的响应，就需要新增一个start_requests方法来重新定义响应。

!> 注意：设置新response的方法的名称只能是start_requests，不能随意命名。

```python
import scrapy

class ChongSpider(scrapy.Spider):
    name = 'chong'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']
    
    # 新增start_requests方法
    def start_requests(self):
        # 定义新的url
        url = 'https://www.....com'
        # 调用parse方法
        yield scrapy.Request(url=url, callback=self.parse)

    # parse方法被调用时，上面start_requests的url请求就完成了，response就是这个url的响应，就不再是start_urls的响应。
    def parse(self, response):
        pass
```

##### 设置请求头

方法一：在 settings.py 里面加一行 USER_AGENT 的定义即可

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36'
```

方法二：借助Downloader Middleware(下载中间件)，在 middlewares.py 添加一个类

```python
import random

# 定义一个类
class RandomUserAgentMiddleware():
    # __init_()方法中定义了三个不同的User-Agent
    def __init__(self):
        self.user_agents = [
            'Mozilla/5.0 (Windows; U; Win 9x 4.90; ja-JP; rv:0.9.4) Gecko/20011128 Netscape6/6.2.1',
            'Mozilla/5.0 (Windows; U; Windows NT 5.1; ja-JP; rv:0.9.4.1) Gecko/20020508 Netscape6/6.2.3',
            'Mozilla/5.0 (Macintosh; N; PPC; ja-JP; macja-pub12) Gecko/20001108 Netscape6/6.0',
        ]
    
    # 使用前面提到的process_request()方法，可以直接修改参数request的属性
    def process_request(self, request, spider):
        # 设置request的User-Agent属性，随机选择的User-Agent
        request.headers['User-Agent'] = random.choice(self.user_agents)
```

Downloader Middleware(下载中间件)写好以后，我们还要去调用它。在 settings.py 中，将 `DOWNLOADER_MIDDLEWARES` 取消注释，添加上面的类：

```python
# test1项目名称，middlewares即middlewares.py文件，RandomUserAgentMiddleware上面定义的类
DOWNLOADER_MIDDLEWARES = {
   'test1.middlewares.RandomUserAgentMiddleware': 543,
}
```

##### 设置代理IP

在项目文件的 middlewares.py 文件，新建 `ProxyMiddleware` 类，添加代理IP:

```python
import random

# 添加代理IP中间件
class ProxyMiddleware(object):
    # __init_()方法中添加了三个代理IP
    def __init__(self):
        self.proxy = [
            '111.160.169.54:41820',
            '115.218.214.43:9000',
            '220.249.149.10:9999',
        ]
    
    # 使用前面提到的process_request()方法，可以直接修改参数request的属性
    def process_request(self, request, spider):
    	# 随机获取代理IP
        request.meta['proxy'] = 'http://'+random.choice(self.proxy)
```

在settings文件取消 `DOWNLOADER_MIDDLEWARES` 注释，并添加上面代理IP的类方法：

```python
# test1项目名称，middlewares即middlewares.py文件，ProxyMiddleware上面定义的类
DOWNLOADER_MIDDLEWARES = {
    'test1.middlewares.ProxyMiddleware': 543,
}
```

##### 设置断点续爬

在爬虫运行过程中，可能由于网络问题、或者突然断电等多种原因，导致爬虫意外结束。如果重启爬虫，前面爬取的数据又需要重新爬取一次，浪费时间；若舍弃没有爬取的数据，就会造成数据的不完整。**因此最好的办法就是将已经爬取的url保存起来，实现断点续爬。**

```python
import redis
import scrapy

class ChongSpider(scrapy.Spider):
    # 使用Redis数据库的集合类型set来保存url
    con = redis.StrictRedis(host="127.0.0.1",port=6379)

    def parse(self, response):
        # 将已下载页面的url存入Redis的save_url中
        self.con.sadd('save_url',response.request.url)
        ...
        # 如果新构造的url不存在在Redis的save_url中，再执行parse方法
        if not self.con.sismember('save_url', url):
            yield scrapy.Request(url=url, callback=self.parse, dont_filter=True)
```

##### 添加Item处理

**如果想对结果进行处理，则可以在 pipelines.py 定义 `Item Pileline` 来实现，定义一个类并实现 `process_item()` 方法即可。启用 `Item Pipeline` 后会自动调用这个方法。`process_item()` 方法必须返回包含数据的字典或 Item 对象，或者抛出 `Dropltem` 异常。**

![QQ截图20200816021348](image/QQ截图20200816021348.png)

在项目文件的 pipelines.py 文件，新建 `Test1Pipeline` 类，添加处理方法:

```python
# 引入DropItem
from scrapy.exceptions import DropItem

class Test1Pipeline:
    def __init__(self):
        # 设置筛选长度50
        self.limit = 50

    # 参数item：每次Spider生成的Item都会作为参数传递过来
    # 参数spider：就是Spider实例
    def process_item(self, item, spider):
        if item['text']:
            if len(item['text']) < self.limit:
                # 超过50长度的字符用...代替
                item['text'] = item['text'][0:self.limit].rstrip()+'...'
            return item
        else:
            DropItem('Miss Text')
```

在settings文件取消 `ITEM_PIPELINES` 注释，并添加上面去重的类方法：

```python
# test1项目名称，pipelines即pipelines.py文件，DuplicatesPipeline上面定义的类
DOWNLOADER_MIDDLEWARES = {
    'test1.pipelines.Test1Pipeline': 300,
}
```

##### 添加Item去重

假如 Item 有一个 `id` 属性，且属性值是唯一的，但是我们的 spider 返回的多个 Item 中包含有相同的 id，这时就要在 pipeline(项目管道) 中添加一个去重功能了，丢弃那些已经被处理过的item。

在项目文件的 pipelines.py 文件，新建 `DuplicatesPipeline` 类，添加去重方法:

```python
# 导入DropItem异常
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):
    def __init__(self):
        # 这里定义一个空集合，因为集合能自动去重
        self.ids_seen = set()

    def process_item(self, item, spider):
        # 判断Item的id是否再集合中
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
```

在settings文件取消 `ITEM_PIPELINES` 注释，并添加上面去重的类方法：

```python
# test1项目名称，pipelines即pipelines.py文件，DuplicatesPipeline上面定义的类
DOWNLOADER_MIDDLEWARES = {
    'test1.pipelines.DuplicatesPipeline': 300,
}
```

##### 对接Selenium

**Scrapy 抓取页面的方式和 requests 库类似，都是直接模拟 HTTP 请求，但 Scrapy 也不能抓 JavaScript 动态渲染的页面。**如果 Scrapy 可以对接 Selenium ，那 Scrapy 就基本可以处理任何网站的抓取了。

因为 Selenium 的方法名称和使用方式没有发生变化，这就不具体讲解了，只是列出常用方法，这些方法大多在Middleware(中间件)中使用：

```python
# middlewares.py文件
from selenium import webdriver
from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from scrapy.http import HtmlResponse 
from logging import getlogger

class SeleniumMiddleware():
    # 对象初始化
    def __init__(self, timeout=None, service_args=[]):
        self.logger = getLogger(__name__)
        self.timeout =timeout
        # 用PhantomJS抓取页面
        self.browser = webdriver.PhantomJS(service_args=service_args)
        # 窗口大小(1400, 700)
        self.browser.set_window_size(1400, 700)
        # 页面加载超时时间
        self.browser.set_page_load_timeout(self.timeout)
        self.wait = WebDriverWait(self.browser, self.timeout)

    def del_(self):
        self.browser.close()

    def process_request(self, request, spider):
        """
        :param request: Request 对象
        :param spider: Spider 对象
        :return: HtmlResponse
        """
        # 内容省略
        # 最后，页面加载完成之后，调用PhantomJS的page_source属性即可获取当前页面的源代码，然后用它来直接构造并返回一个HtmlResponse对象。构造这个对象的时候需要传入多个参数，如url、body等，这些参数实际上就是它的基础属性。
        return HtmlResponse(url=request.url, body=self.browser.page_source, request=request, encoding='utf-8', status=200)
    
    @classmethod
    def from_crawler(cls, crawler):
        # 内容省略
```

在 settings.py 里，我们设置调用刚才定义的 `SeleniumMiddleware`，如下所示：

```python
DOWNLOADER_MIDDLEWARES = {
   'test1.middlewares.SeleniumMiddleware': 543,
}
```





