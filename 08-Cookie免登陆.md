# Cookie免登陆

##### Cookie

  使用浏览器在浏览网页的过程中，比如登录一个页面。我们经常会在此时设置30天内记住我，或者自动登录选项。那么它们是怎么记住我的呢？答案就是cookie了。

  Cookie是由HTTP服务器设置的，保存在浏览器中，即本地磁盘中。

  Cookies是服务器在本地机器上存储被访问网站相关信息的小段文本，并随每一个请求发送至同一服务器，以便访问时减少一些步骤，是在客户端保持状态的方案。
  
  Cookies是字典的形式存储的，主要内容包括：名字，值，过期时间，路径和域。

**注意：Cookies并不是永久有效的，是有有效期的，Cookies的有效取决于服务器端对应的Session是否被销毁。**

##### Session

  Session就是服务器为访问用户所创建并维护的一个对象，是存在服务器的一种用于存放数据，在服务器创建对象的同时，会为该对象产生一个唯一的编号，这个编号称为sessionID。
  
  服务器以cookie的方式将sessionID存放在客户端中，当浏览器再次访问该服务器的时候，会将sessionID作为cookie信息带服务器，服务器可以通过该sessionID检索到以前的session对象，再让其访问。

  当用户在Web页间跳转时，Session对象中的变量不会丢失而是在整个用户会话中一直存在下去。一般这个值会有个时间限制，超时后毁掉这个值，默认30分钟。

##### 完整会话

  登录网站时，会在客户端生成 Cookies ，而 Cookies 里面保存了 SessionID 的信息， 登录之后的后续请求都会携带生成后的 Cookies 发送给服务器。服务器就会根据 Cookies保存的SessionID查找出对应的Session对象，进而找到会话。如果当前Cookie是有效的，那么服务器就判断用户当前已经登录了，返回请求的页面信息，这样我们就可以看到登录之后的页面；如果是无效的就会返回登陆页面。

### 免登陆原理

  既然服务器是通过Cookies来判断用户是否登录，我们这里就可以获取登录之后的 Cookies ，来实现免登录。
  
  requests方法：手动输入账号密码登录——谷歌浏览器（开发模式）——Network——Doc(Cookies是文本)——复制Cookies——headers中添加Cookie——访问登陆后的页面。
  
  selenium方法：输入账号密码登录——浏览器对象.get_cookies()——获取Cookies——循环取出Cookies中的Cookie——循环添加Cookie——浏览器对象.add_cookies(Cookie)——访问登录后页面。

**注意：存储Cookies时，最好也存储获取Cookies的时间，根据时间的差值，来判断当前Cookies是否有效。**

##### requests方法

  这里以人人网为例，获取用户的个人主页，就需要获取用户登录Cookies来实现免登录，获取cooike最简单的方法就是在**登录后个人主页面（我的主页）**的Network中Doc中的profile文件里面获取cookie，再将其复制添加到headers中

```
import requests

# 构造headers请求头
headers = {
		  'user-agent': 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko)Chrome/63.0.3239.132 Safari/537.36 ',
           'Cookie': 'anonymid=jql8x1c0193wle; depovince=SC; _r01_=1; JSESSIONID=abczJ1Z8-EXoR59aMEKGw; ick_login=9126b72a-1472-4dfe-a053-6979c002ca98; id=969371812; ver=7.0; loginfrom=null; jebe_key=8bb72614-531a-4f0b-bd64-c20cba0c6322%7C3bd259db9dc13f144c59fb55991d52a0%7C1546800238259%7C1%7C1546800241523; wp_fold=0; t=eda27961ff884ffe91b192ee390fa41c2; societyguester=eda27961ff884ffe91b192ee390fa41c2; xnsid=d937ecd7; jebecookies=665d4da8-0a81-4092-9453-719c1914469c|||||'
           }

# 获取网页代码
url = 'http://www.renren.com/969371812/profile'
reponse = requests.get(url=url, headers=headers)

# 这里的utf-8是对响应的内容进行编码
reponse.encoding = 'utf-8'
print(reponse.text)
```

##### selenium方法

```
# 先清空cookies
browser.delete_all_cookies()
# 访问登录页面
browser.get('...')
# 输入账号密码
browser.find_element_by_id('username').send_keys('username')
browser.find_element_by_id('password').send_keys('password')
# 点击登录按钮
browser.find_element_by_xpath('...').click()
# 获取登录后的cookies，将Cookie进行保存
cookies = browser.get_cookies()
###################################################################
# 当在次访问需要登录的页面时，就加载保存的cookie实现免登录
# 清空cookie
browser.delete_all_cookies()
# 获取到上面保存的cookies内容：[{'domain': '.suning.com', 'expiry': 1557537894...}...]
# 因为cookie内容是列表里面套字典，因此需要循环来添加cookie
for i in range(len(cookies)):
    browser.add_cookie(cookies[i])
# 访问页面
browser.get('...')
```
