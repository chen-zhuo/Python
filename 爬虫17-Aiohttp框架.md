# Aiohttp框架

关于aiohttp框架的文档参看：[aiohttp官方文档](https://docs.aiohttp.org/en/stable/)

得益于多线程能解决很多IO阻塞问题这一特性，多线程爬虫比单线程爬虫，在效率上有了很大的提高，但由于**我们所用的requests库是一个同步库，也就是说从向服务器请求网页到服务器响应并返回网页这一过程是同步的，如果在下载网页时间太长会导致阻塞，而且CPU性能和网络资源在这过程中也得不到充分的使用。**

对比前面多线程例子，协程的优势在于：

1. **一个线程中，一个协程阻塞时，马上切换到另一个协程，提升CPU利用率，用协作的方式加速程序执行。**
2. **相比于多线程，内存开销更小，自由切换，避免了无意义的调度。**
3. **因为只有一个线程，不需要锁机制。**

**aiohttp是一个基于asyncio和部分C语言实现的HTTP框架，它可以帮助我们异步地实现HTTP请求，也就是说，我反正只管去发送网页的请求，不管服务器是否已经返回数据，这样使得我们的程序效率大大提高。** 

?> 提示：aiohttp只支持3.5.3以后的Python版本。

## 基本使用

### 快速安装

我们之前使用的 `requests` 三方库并不支持异步 I/O，如果要使用异步 I/O 的方式来加速爬虫代码的执行，可以安装和使用名为 `aiohttp` 的三方库。执行下面命令：

```bash
pip install aiohttp
```

### 请求方式

GET请求：

```python
# 引入一个类aiohttp.ClientSession建立一个session对象，用session对象去打开网页
async with aiohttp.ClientSession() as session:
    # session.get(url)即GET请求
    async with session.get(url) as response:
        # await的返回值就是协程运行的结果
        html = await response.text()
```

POST请求：

```python
# 提交普通参数
async with aiohttp.ClientSession() as session:
    # session.post(url,data)即POST请求，data即POST表单提交的参数
    async with session.post(url, data=data) as response:
        # await的返回值就是协程运行的结果
        html = await response.text()

# 提交json参数
async with aiohttp.ClientSession() assession:
    res = await session.post('url', json={'key': 'value'})
        # 当使用aiohttp时，只有在awiat resp.json() 时才会真正发送请求。
        result = await res.json()
```

### 设置代理

requests设置代理的方式：`proxy的复数形式`

```python
proxies = {
            'http': 'http://124.113.251.151:8085',
            'https': 'https://124.113.251.151:8085',
        }
response = requests.get('URL', proxies=proxies)
```

aiohttp设置代理的方式：`proxy的单数形式`

```python
async with aiohttp.ClientSession() as session:
    proxy = "http://124.113.251.151:8085"
    async with session.get(url, proxy=proxy) as response:
        # await的返回值就是协程运行的结果
        html = await response.text()
```

!> 注意：aiohttp不支持https代理。

## 异步爬虫

### 使用案例

下面的代码使用 `aiohttp` 抓取了 `10` 个网站的首页并解析出它们的标题：

```python
import re
import aiohttp
import asyncio
from aiohttp import ClientSession

# 请求头
headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36'}

# 正则表达式
TITLE_PATTERN = re.compile(r'<title.*?>(.*?)</title>', re.DOTALL)

# fetch_page_title是一个被async关键字修饰的异步函数，调用该函数会获得协程对象，
async def fetch_page_title(url):
    # 创建ClientSession对象，跟requests中的Session对象并没有本质区别
    async with aiohttp.ClientSession(headers=headers) as session:
        # 通过它的get方法向指定的URL发起请求
        async with session.get(url, ssl=False) as resp:
            if resp.status == 200:
                # await会让因为I/O操作阻塞的子程序放弃对CPU的占用，使其他子程序运转起来去抓取页面
                html_code = await resp.text()
                # 使用正则表达式捕获组操作解析网页标题
                matcher = TITLE_PATTERN.search(html_code)
                title = matcher.group(1).strip()
                print(title)

def main():
    urls = [
        'https://www.python.org/',
        'https://www.jd.com/',
        'https://www.baidu.com/',
        'https://www.taobao.com/',
        'https://git-scm.com/',
        'https://www.sohu.com/',
        'https://gitee.com/',
        'https://www.amazon.com/',
        'https://www.usa.gov/',
        'https://www.nasa.gov/'
    ]
    objs = [fetch_page_title(url) for url in urls]
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.wait(objs))
    loop.close()


if __name__ == '__main__':
    main()

'''
输出：
京东(JD.COM)-正品低价、品质保障、配送及时、轻松购物！
搜狐
淘宝网 - 淘！我喜欢
百度一下，你就知道
Gitee - 基于 Git 的代码托管和研发协作平台
Git
NASA
Official Guide to Government Information and Services   &#124; USAGov
Amazon.com. Spend less. Smile more.
Welcome to Python.org
解释：从上面的输出可以看出，网站首页标题的输出顺序跟它们的 URL 在列表中的顺序没有关系。
'''
```

下面的代码使用 `aiohttp` 抓取了豆瓣电影Top250并解析出它们的详细信息：

```python
'''
网址：爬取豆瓣电影Top250
目的：测试aiohttp爬取250个URL
'''
import re
import time
import asyncio
import aiohttp
import requests
from fake_useragent import UserAgent

# 电影详情网页
# async标记为协程对象
async def parse(url):
    # 引入一个类aiohttp.ClientSession建立一个session对象，用session对象去打开网页
    async with aiohttp.ClientSession() as session:
        # session.get(url)即GET请求，session.post(url,data)即POST请求，
        async with session.get(url) as response:
            # await的返回值就是协程运行的结果
            m_webcode = re.sub(r'\s*', '', await response.text())
            # 获取电影名字、评分、导演、语言
            try:
                m_name = re.search(r'name="title"value="(.*?)"', m_webcode).group(1)
                m_score = re.search(r'property="v:average">(.*?)<',m_webcode).group(1)
                m_director = re.search(r'rel="v:directedBy">(.*?)<', m_webcode).group(1)
                m_language = re.search(r'语言:</span>(.*?)<br/>', m_webcode).group(1)
                print(m_name, m_score, m_director, m_language)
            except:
                print('页面不存在')

if __name__ == '__main__':
    start_time = time.time()
    m_urls = []
    # 获取全部电影网页url
    urls = ['https://movie.douban.com/top250?start=%d&filter='%(i*25) for i in range(0,10)]
    for url in urls:
        headers = {'User-Agent': UserAgent(use_cache_server=False).chrome}
        response = requests.get(url, headers=headers)
        webcode = re.sub(r'\s*', '', response.text)
        for item in re.findall(r'<divclass="item">.*?</div>', webcode):
            m_href = re.search(r'href="(.*?)"', item).group(1)
            m_urls.append(m_href)
    # 创建事件循环
    loop = asyncio.get_event_loop()
    # asyncio.ensure_future创建任务
    tasks = [asyncio.ensure_future(parse(m_url)) for m_url in m_urls]
    # asyncio.gather创建协程对象，列表task加*作用，将列表解开成每个独立的元素参数，传入函数
    tasks = asyncio.gather(*tasks)
    # 运行事件循环
    loop.run_until_complete(tasks)
    end_time = time.time()
    print(f'花费时间：{end_time-start_time}')
'''
输出：
页面不存在（'电影"V字仇杀队"网页失效'）
...
消失的爱人GoneGirl(2014) 8.7 大卫·芬奇 英语
花费时间：10.334077835083008（爬取三次取平均值）

解释1：从结果输出顺序可以看出，协程对象创建顺序不同于输出顺序，这是由于并发程序本身执行顺序不确定性造成的。
解释2：相比于多线程爬虫，异步爬虫性能略优，因为没有了线程间的来回切换再加上aiohttp异步请求优势。
'''
```

?> 详细测评参看：[浅度测评：requests、aiohttp、httpx 我应该用哪一个？](https://blog.csdn.net/u010467643/article/details/106132125)

### 高并发异常

上面的例子中，测试aiohttp爬取250个URL的效率，但如果URL的超过2000个，程序会报错：`ValueError: too many file descriptors in select()`。报错的原因字面上看是 Python 调取的 select 对打开的文件有最大数量的限制，这个其实是操作系统的限制，**linux打开文件的最大数默认是1024，windows默认是509，超过了这个值，程序就开始报错。**这里我们有**三种方法解决**这个问题：

**1.限制并发数量**。（一次不要塞那么多任务，或者限制最大并发数量）

**2.使用回调的方式**。

**3.修改操作系统打开文件数的最大限制，在系统里有个配置文件可以修改默认值，具体步骤不作说明了。**

个人推荐限制并发数的方法，设置并发数为500，处理速度更快。可以将上面代码修改为：

```python
import re
import time
import asyncio
import aiohttp
import requests
from fake_useragent import UserAgent

# 电影详情网页
# async标记为协程对象
async def parse(url, semaphore):
    async with semaphore:
        # 引入一个类aiohttp.ClientSession建立一个session对象，用session对象去打开网页
        async with aiohttp.ClientSession() as session:
            # session.get(url)即GET请求，session.post(url,data)即POST请求，
            async with session.get(url) as response:
                # await的返回值就是协程运行的结果
                m_webcode = re.sub(r'\s*', '', await response.text())
                # 获取电影名字、评分、导演、语言
                try:
                    m_name = re.search(r'name="title"value="(.*?)"', m_webcode).group(1)
                    m_score = re.search(r'property="v:average">(.*?)<',m_webcode).group(1)
                    m_director = re.search(r'rel="v:directedBy">(.*?)<', m_webcode).group(1)
                    m_language = re.search(r'语言:</span>(.*?)<br/>', m_webcode).group(1)
                    print(m_name, m_score, m_director, m_language)
                except:
                    print('页面不存在')

if __name__ == '__main__':
    start_time = time.time()
    m_urls = []
    # 获取全部电影网页url
    urls = ['https://movie.douban.com/top250?start=%d&filter='%(i*25) for i in range(0,10)]
    for url in urls:
        headers = {'User-Agent': UserAgent(use_cache_server=False).chrome}
        response = requests.get(url, headers=headers)
        webcode = re.sub(r'\s*', '', response.text)
        for item in re.findall(r'<divclass="item">.*?</div>', webcode):
            m_href = re.search(r'href="(.*?)"', item).group(1)
            m_urls.append(m_href)
    # 创建事件循环
    loop = asyncio.get_event_loop()
    # 限制并发量为500
    semaphore = asyncio.Semaphore(500)
    # 封装多个协程
    tasks = [parse(m_url, semaphore) for m_url in m_urls]
    # 运行事件循环
    loop.run_until_complete(asyncio.gather(*tasks))
    end_time = time.time()
    print(f'花费时间：{end_time-start_time}')
```

### 总结

我们可以得出一个结论，使用多线程和异步 I/O 都可以改善爬虫程序的性能，因为我们不用将时间浪费在因 I/O 操作造成的等待和阻塞上。另外，在运行时间差别不大的情况下，多线程的代码比异步 I/O 的代码耗费了更多的 CPU 资源，这是因为多线程的调度和切换也需要花费 CPU 时间。至此，三种方式在 I/O 密集型任务上的优劣已经一目了然，当然这只是在我的电脑上跑出来的结果。如果网络状况不是很理想或者目标网站响应很慢，那么使用多线程和异步 I/O 的优势将更为明显，有兴趣的读者可以自行试验。
