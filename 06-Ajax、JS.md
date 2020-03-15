# Ajax数据

### 认识Ajax

##### Ajax

​	Ajax数据异步加载，**即异步的 JavaScript 和 XML ，它不是一门编程语言，而是利用 JavaScript 在保证页面不被刷新、页面链接不改变的情况下与服务器交换数据并更新部分网页的技术。**（网页局部刷新技术）

​	**原始的页面最初不会包含某些数据，原始页面加载完后，会再向服务器请求某个接口获取数据，然后数据才被处理从而呈现到网页上，这其实就是发送了一个 Ajax 请求。**

​	例如：百度翻译，在汉字输入的过程中，旁边的英文也在不断更新，而网页并没有全部刷新。

##### 查看Ajax流程

1. 打开含有Ajax的网页
2. 按F12打开’开发者工具‘，
3. **点击选项卡’Network‘（刷新一下页面，查看浏览器和服务器所有的请求和响应）**
4. **在点击下方的选项卡’XHR‘（这里可以将请求里属于Ajax请求给过滤出来）就可以查看Ajax。**
5. 点击文件中的Headers选项，观察文件的详细信息，**其中 Request Headers 的一个信息为 X-Requested-With:XMLHttpRequest ，这就标记了此请求是 Ajax 请求**。
6. **再点击Response选项，这里就显示了文件通过Ajax请求的内容。**

##### 发展趋势

​	Web 发展的趋势来看，这种形式的页面越来越多。**网页的原始 HTML 文档不会包含任何数据，数据都是通过Ajax统一加载后再呈现出来的，这样在 Web 开发上可以做到前后端分离，而且降低服务器直接渲染页面带来的压力。**	

### 解决方案

##### 模拟 Ajax 请求

​	有的页面是通过JavaScript渲染的，直接利用 requests 等库来抓取原始页面，是无法获取到有效数据的，这时需要**分析网页后台向接口发送的 Ajax 请求，再利用 requests 来模拟 Ajax 请求，那么就可以成功抓取了**。

```
# 导入request库、lxml解析库
import requests
from lxml import etree

# url不再是网页地址，而是Ajax请求的地址（XHR中文件的Headers选项中的Request URL）
url = 'https://www.qq.com/ninja/news_sichuan_2018.htm'

# 构造请求头
headers = {
        'Referer': 'https://www.qq.com/',
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
        'X-Requested-With': 'XMLHttpRequest',
        }
response = requests.get(url=url, headers=headers)

# 打印Ajax请求的响应
print(response)

# 打印Ajax请求的响应内容
print(response.text)

# lxml解析网页内容
response1 = etree.HTML(response.text)
# xpath选择器提取网页文本（text()）内容
title = response1.xpath('//*[@class="yw-list"]/li/a/text()')
# 打印内容
print(title)

输出：
<Response [200]>

# Ajax请求的内容
<ul class="yw-list" bosszone="dfz_1">
          <li><a href="http://cd.qq.com/a/20181124/002042.htm" target="_blank">四川人注意啦！这12批食品抽检不合格</a></li>
          <li><a href="http://cd.qq.com/a/20181124/002147.htm" target="_blank">成都小区连续四个月“天降狗屎” 肇事者称因压力大</a></li>...
  
# xpath提取的标签文本内容
['四川人注意啦！这12批食品抽检不合格', '成都小区连续四个月“天降狗屎” 肇事者称因压力大'...]
```

##### selenium工具

**selenium是一款自动化测试工具，它通过驱动浏览器来模拟用户操作，获取浏览器当前呈现的页面的源代码，做到可见即可爬。**

```
import time
# 从selenium导入webdriver
from selenium import webdriver
# 通过webdriver驱动谷歌浏览器将此方法再放入变量driver
driver = webdriver.Chrome()
# driver.get方法访问url
driver.get('https://www.qq.com/')
# 休息5秒
time.sleep(5)
# driver.page_source方法获取网页代码
print(driver.page_source)

输出：
...
<ul class="yw-list" bosszone="dfz_1">
          <li><a href="http://cd.qq.com/a/20181124/002042.htm" target="_blank">四川人注意啦！这12批食品抽检不合格</a></li>
          <li><a href="http://cd.qq.com/a/20181124/002147.htm" target="_blank">成都小区连续四个月“天降狗屎” 肇事者称因压力大</a></li>...
```

**注意：Selenium 中， get （）方法会在网页框架加载结束后结束执行，此时获取 page_source ，可能并不是浏览器完全加载完成的页面，如有额外的 Ajax 请求，在网页源代码中也不一定能成功获取到。所以需要延时等待，确保节点已经加载出来。**

# JS加密

### 简介

##### JS加密

**JS是运行在浏览器里面的一种前端语言。**

JS可以对一些前端参数进行加密，以便服务器后台便于验证。如对用户名和密码进行加密，后端可以判断是否是正确的请求还是恶意的攻击。

**JS加密大多出现在表单提交过程中。**

##### 加密算法

经过JS加密传输的就是密文，上面提到 JS是运行在浏览器里面的，**也就是说加密的过程一定是在浏览器完成，也就是一定会把JS代码暴露给使用者，但是有的公司，为了防止被竞争对手抓取或使用自己的代码，就会对JS混淆加密，来达到代码保护。**

通过阅读加密算法，就可以模拟出加密过程，从而达到破解。

常见的加密算法：MD5加密（不可逆，生成32位字符串）、Base64加密（可逆）和shal加密。

### 破解JS加密

在做python爬虫的时候经常会遇到许多的反爬措施，JS加密就是其中一种。

##### 加密cookie

为了防止别人爬取网站里面的内容，使用JS加密cookie就可以区别你是浏览器正确的请求还是使用诸如Httpclient等框架在爬取网页内容了。

**普通的爬虫框架只是负责拿着你给的URL传递一些参数来获取网页源码，它们不能运行JS，也就无法对服务器需要验证的参数进行加密，从而导致服务器会拒绝你的请求，给你返回一个不是你所期望的网页内容的页面**。

![加密JS请求过程](https://img-blog.csdnimg.cn/20190120142936282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyNDgzMTQ1,size_16,color_FFFFFF,t_70)

##### 破解方法

1. 直接驱动浏览器抓取数据，无视js加密。
2. 找到本地加密的js代码，使用python的相关库直接运行js代码。
3. 找到本地加密的js代码，理清加密逻辑，然后用python代码来模仿js代码的流程，生成我们想要的加密的数据。

### 浏览器破解

Selenium：基于真实的浏览器自动化测试框架，可以运行JS。

Selenium会打开本地的谷歌或火狐等浏览器进行操作，浏览器可以运行加密的JS，那么该框架也可以运行加密过的JS从而得到需要的参数，但要是开几十个浏览器窗口去获取网页源码，作为爬虫确实比较慢了。

##### requests方法

先用requests方法来获取网页代码，看看是什么内容。

```
import requests
from fake_useragent import UserAgent

# 网站url
url = 'https://www.hapag-lloyd.cn/zh/home.html#hal-map'
# 请求头
headers = {'user-agent':UserAgent().random}

response = requests.get(url=url,headers=headers)
print(response.text)

输出：
<!DOCTYPE html>
<html><head>
# 下面就是未加载JS的数据，从中我们获取不到任何有用的信息
...
window.kGv=!!window.kGv;try{(function(){(function LJ(){var L=!1;function z(L){for(var s=0;L--;)s+=_(document.documentElement,null);return s}function _(L,s){var z="vi";s=s||new I;return oJ(L,function(L){L.setAttribute("data-"+z,s.OS());return _(L,s)},null)}function I(){this.iz=1;this.SO=0;this.Ll=this.iz;this.SL=null;this.OS=function()
...
</html>
```

##### selenium方法

再用selenium方法来获取网页代码，看看是什么内容。

```
import time

from selenium import webdriver
# 导入chrome选项
from selenium.webdriver.chrome.options import Options

# 建立Option类对象
chrome_options = Options()

# '--headless'设置为无头模式
chrome_options.add_argument('--headless')

# 启用配置（上面的配置不启动，就无效）
browser = webdriver.Chrome(chrome_options=chrome_options)

# get方法访问传入url
browser.get('https://www.hapag-lloyd.cn/zh/home.html#hal-map')

# 因为运行JS也需要短暂时间，休息2秒
time.sleep(2)

# page_source方法输出网页的源代码
print(browser.page_source)

# 关闭浏览器
browser.close()

输出：
<!DOCTYPE html>
<html><head>
# 这里面就有我们需要的信息
...                                    
                            冷藏货物
                          </a>
                        </li>                  
                        <li class="hal-navigation-item">
                          <a href="/zh/products/cargo/dg/safety-first.html" class="hal-rtl--alt" btattached="true">                            
                            危险货物
                          </a>
                        </li>                     
                        <li class="hal-navigation-item">
                          <a href="/zh/products/cargo/special.html" class="hal-rtl--alt" btattached="true">
...
</html>
```

### Python库破解

##### Node.js

在使用这个库之前请确保你的机器上安装了以下其中一个JS运行环境:

- JScript
- JavaScriptCore
- Nashorn
- Node
- PhantomJS
- PyV8
- SlimerJS
- SpiderMonkey

PyExecJS 库会按照优先级调用这些引擎来实现 JavaScript 执行，这里推荐安装 Node.js。

下载地址：https://nodejs.org/en/download/

一路next，即可安装好。

检测PATH环境变量是否配置了Node.js，点击开始=》运行=》输入"cmd" => 输入命令"path"，查看环境变量中是否有nodejs。

输入：node --version，检查Node.js版本

![node-version-test](https://www.runoob.com/wp-content/uploads/2014/03/node-version-test.png)

##### PyExecJS

PyExecJS 是一个可以使用 Python 来模拟运行 JavaScript 的库。大家可能听说过 PyV8，它也是用来模拟执行 JavaScript 的库，可是由于这个项目已经不维护了，而且对 Python3 的支持不好，而且安装出现各种问题，所以这里选用了 PyExecJS 库来代替它。

首先我们来安装一下这个库：

```
pip install PyExecJS
```

运行代码检查一下运行环境：

```
import execjs
print(execjs.get().name)

输出：
Node.js (V8)
```
