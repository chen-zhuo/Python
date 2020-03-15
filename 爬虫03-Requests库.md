# Requests库

参考内容：[requests中文文档](http://2.python-requests.org/zh_CN/latest/)

### 初识Requests

##### requests简介

requests库是python实现的最简单易用的HTTP库，也是写爬虫必须要掌握的库。

##### 安装requests库

requests 属于第三方库，就是 Python 默认不会自带的库，所以我们需要手动安装。

```
pip install requests
```

### 发送请求

##### 请求类型

前面提到过，HTTP有多种请求类型，requests库同样可以实现。**一般在爬虫中用的最多的是get请求、post请求。**

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

**当你在浏览器中输入一个网址并回车访问时，就是一个get请求**。

在requests中也只需要输入一个网址就可以发送get请求，重要参数有两个：

1. **url参数：HTTP请求必备的参数，作用是确定访问的网址。**
2. **params参数：以字典的形式传递url中参数。**

```python
import requests

# 基本的GET请求
# 方式一：网址传递给变量，在传递给url参数
link = 'https://httpbin.org/get'
response = requests.get(url=link)
# 方式二：直接传递给url参数（推荐）
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

# 方式一：url包含键值对参数
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

post请求和get请求的主要区别在于：

1. **post请求传递的参数长度没有限制；get请求则有所限制。**
2. **post请求传递的参数形式是通过表单传递的，不会暴露在url当中，例如：用户密码登录用的就是post请求；get请求传递的参数是以键/值对的形式跟在一个问号的后面的。**

post请求也很简单，主要是网址和传参：

1. **url参数：这里和get请求都一样，每个HTTP请求都必须要有的一个参数。**
2. **data参数：以字典`dict`的形式提交参数时，若不指定`content-type`，默认为`application/x-www-form-urlencoded`，相当于普通form表单提交；以字符串`str`的形式提交参数时，若不指定`content-type`，默认为`application/json`。**
3. **json参数：以json数据的结构形式提交参数，若不指定`headers`中的`content-type`，默认为`application/json`。**
4. **用data参数提交数据时，`request.body`的内容则为`a=1&b=2`的这种形式，用json参数提交数据时，`request.body`的内容则为'{"`a": 1, "b": 2}'`的这种形式**

```python
import json
import requests

dict1 = {
    'name': 'germey',
    'age': 22
}
# 直接以字典的格式传递数据
response = requests.post('https://httpbin.org/post', data=dict1)
print(response.text)
'''
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

# 解释：因为post中data参数默认是表单传递参数，因此传递的参数出现在了form表单里面。
'''

# 将python字典dict类型数据自动转化为json数据化传递
response = requests.post('https://httpbin.org/post', json=dict1)
print(response.text)
'''
输出：
{
  "args": {}, 
  "data": "{\"name\": \"germey\", \"age\": 22}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "29", 
    "Content-Type": "application/json", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.19.1", 
    "X-Amzn-Trace-Id": "Root=1-5e57ebd1-d0d9017460a1ecf0e4dca6dc"
  }, 
  "json": {
    "age": 22, 
    "name": "germey"
  }, 
  "origin": "180.97.206.96", 
  "url": "https://httpbin.org/post"
}

解释：因为post中json参数默认是json格式传递参数，因此出现在了"json"里面。
'''
```

##### 添加请求头

没有请求头的爬虫是没有灵魂的爬虫，结合之前讲过请求头中最重要的一个参数User-Agent和请求头库，可以写一个基本的爬虫了。

```python
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 图片地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)
```

### 接受响应

当我们向服务器发送请求后，服务器会返回给客户端响应，在响应中包含许多内容，通过响应的各种属性可以轻松获取到我们想要的内容。

##### 响应内容属性

```python
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 图片地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)

# 打印响应状态 
print(response)					# <Response [200]>，表示获取到响应
# 打印网页响应的状态码
print(response.status_code)    	# 200，200表示成功访问
# 设置响应内容的编码
# Requests会基于HTTP头部响应的编码对网页文本编码作出推测，你若设置了encoding编码，后面都使用新编码。
response.encoding = 'UTF-8'          # 主要针对于网页中文乱码的情况
response.encoding = 'GBK'            # 主要针对于网页编码为gbk、gb2312类型的内容
response.encoding = 'unicode_escape' # 主要针对于网页编码为unicode类型的的内容
# 以字符型文本形式打印响应内容
print(response.text)			# 主要用于打印网页的代码和文本内容
# 以二进制的形式打印响应内容数据
print(response.content)			# 主要用于打印网页中图片、音频、视频（二进制文件）内容
# 以json格式打印响应的内容
print(reponse.json())			# 等价于print(json.loads(reponse.text))
# 打印响应头
print(response.headers)         # 查看服务器返回内容的请求头
# 打印请求头
print(response.requests.headers)# 查看访问时的请求头
# 打印响应的url内容
print(response.url)				
# 打印响应的cookie
print(response.cookies)         # 打印响应内容中Cookie
# 打印响应的cookies.items
print(response.cookies.items()) # 以字典的形式打印响应内容中Cookie
# 遍历cookies内容打印cookie
for key, value in response.cookies.items():
    print(key + '=' + value)
```

##### 爬取网页

爬取基本的网页内容，可以使用下面代码：

```python
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 图片地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)
# 输出网页内容
print(response.text)
```

##### 爬取图片

图片、音频、视频都属于二进制文件，可以使用下面代码：

```python
import requests
from fake_useragent import UserAgent

# 请求头
headers = {'User-Agent': UserAgent().random,}
# 图片地址
url = '...'
# 获取响应
response = requests.get(url=url, headers=headers)

# wb(二进制写入模式)，在当前路径打开或新建image.png文件，重新写入数据。
with open('image.png', 'wb') as f:
# 将response.content(响应图片的二进制流数据)写入到变量f的路径下的文件
    f.write(response.content)
```

