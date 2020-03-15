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
