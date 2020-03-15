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
