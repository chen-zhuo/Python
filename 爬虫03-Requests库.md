# Requests库

### 初识Requests

##### requests简介

requests库是python实现的最简单易用的HTTP库，也是写爬虫必须要掌握的库。

##### 安装requests库

```
pip install requests
```

### 发送请求

##### 请求类型

前面提到过，HTTP有多种请求类型，requests库同样可以实现。一般在爬虫中用的最多的是get请求、post请求。

```python
# 导入requests
import requests

# get请求
requests.get('http://httpbin.org/get')

# post请求
requests.post('http://httpbin.org/post', data = {'key':'value'})

# put请求
requests.put('http://httpbin.org/put', data = {'key':'value'})

# delete请求
requests.delete('http://httpbin.org/delete')
```

##### GET请求

**当你在浏览器中输入一个网址并回车访问时，就是一个get请求**。在requests中也只需要输入一个网址就可以发送get请求。

```python
import requests

# 基本的GET请求
response = requests.get('https://httpbin.org/get')

#打印响应的内容信息
print(response.text)
'''
输出：
{
  "args": {}, 
  "headers": {
    "Accept": "/", 
    "Accept-Encoding": "gzip, deflate", 
    "Connection": "close", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.19.1"
  }, 
  "origin": "182.149.163.126", 
  "url": "https://httpbin.org/get"
}

# 解释：访问https://httpbin.org/get这个网址会返回客户端发送的请求内容。因为没有设置User-Agent，所以默认为python-requests/2.19.1；
'''

# 带参GET请求：在URL中传递的参数会以键/值对的形式跟在一个问号的后面。例如， httpbin.org/get?key=val。这里的params关键字参数，就可以字典形式来传递这些参数。字典里值为 None 的键都不会被添加到 URL 的查询字符串里。

# 方式一：url包含键值对
response = requests.get('https://httpbin.org/get?name=germey&age=22')
# 方式一：使用params关键字参数
data = {'name': 'germey', 'age': 22}
response = requests.get('https://httpbin.org/get', params=data)

print(response.text)
'''
输出：
{
  "args": {
    "age": "22", 
    "name": "germey"
  }, 
  "headers": {
    "Accept": "/", 
    "Accept-Encoding": "gzip, deflate", 
    "Connection": "close", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.19.1"
  }, 
  "origin": "182.149.163.126", 
  "url": "https://httpbin.org/get?name=germey&age=22"
}

# 解释："args"就是传递的参数，里面就显示了url中所带的参数。
'''
```

##### POST请求

```
import requests
data = {
    'name': 'germey',
    'age': 22
}
response = requests.post('https://httpbin.org/post', data=data)
print(response.text)

输出：
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "age": "22", 
    "name": "germey"
  }, 
  "headers": {
    "Accept": "/", 
    "Accept-Encoding": "gzip, deflate", 
    "Connection": "close", 
    "Content-Length": "18", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.19.1"
  }, 
  "json": null, 
  "origin": "182.149.163.126", 
  "url": "https://httpbin.org/post"
}
```

### response属性

```
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,
           'Cookie': '...'}
# 地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)
# 转码
response.encoding = 'utf-8'

# 打印响应 
print(response)					# <Response [200]>，表示获取到响应
# 打印网页响应的状态码
print(response.status_code)    	# 200，200表示成功访问
# 以字符型文本形式打印响应内容
print(response.text)			# 主要用于打印网页的代码和文本内容
# 以二进制的形式打印响应内容数据
print(response.content)			# 主要用于打印网页中图片、音频、视频（二进制文件）内容
# 以json格式打印响应的内容
print(reponse.json())			# 等价于print(json.loads(reponse.text))
# 打印响应头
print(response.headers)
# 打印响应的url内容
print(response.url)				# 就是上面的url
# 打印响应的history内容
print(response.history)
# 打印响应的cookie
print(response.cookies)
# 打印响应的cookies.items
print(response.cookies.items())
# 遍历cookies内容打印cookie
for key, value in response.cookies.items():
    print(key + '=' + value)
```

### 爬取图片

图片、音频、视频都属于二进制文件，可以使用下面代码。

```
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 图片地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)

# wb(二进制写入模式)，在当前路劲打开或新建image.png文件，重新写入数据。
with open('./image.png', 'wb') as f:
# 将response.content(响应图片的二进制流数据)写入到变量f的路径下的文件
    f.write(response.content)
```

### 爬取网页信息

```
import pymysql
import requests
from lxml import etree
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)
# 转码
response.encoding = 'utf-8'


# 解析页面
def analyze():
    info = etree.HTML(response.text)
    # text()获取文本
    user = info.xpath('.../text()')
    # @src获取src属性
    image = info.xpath('.../@src')
    return user, image


# 存入MySQL数据库
def save_mysql(info):
    # 连接数据库
    host = '127.0.0.1'
    user = 'root'
    password = '...'
    database = '...'
    port = 3306
    db = pymysql.connect(host, user, password, database, charset='utf8', port=port)
    # 生成操作游标（对数据库进行操作）
    cursor = db.cursor()
    # sql语句，INSERT INTO插入，table_name数据表名，user,image字段名，values值，%s字符串，info就是上面方法的信息元祖
    sql = "INSERT INTO table_name(user, image) values('%s','%s')" % info
    # 通过操作游标的execute()执行sql语句
    cursor.execute(sql)
    # 提交执行结果（没有这它，数据库存不进数据）
    db.commit()

if __name__ == '__main__':
    info = analyze()
    save_mysql(info)
```

