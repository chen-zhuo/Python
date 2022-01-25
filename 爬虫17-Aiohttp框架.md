### aiohttp库

我们之前使用的`requests`三方库并不支持异步 I/O，如果希望使用异步 I/O 的方式来加速爬虫代码的执行，我们可以安装和使用名为`aiohttp`的三方库。

安装`aiohttp`。

```bash
pip install aiohttp
```

下面的代码使用`aiohttp`抓取了`10`个网站的首页并解析出它们的标题。

```python
import asyncio
import re

import aiohttp
from aiohttp import ClientSession

TITLE_PATTERN = re.compile(r'<title.*?>(.*?)</title>', re.DOTALL)


async def fetch_page_title(url):
    async with aiohttp.ClientSession(headers={
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36',
    }) as session:  # type: ClientSession
        async with session.get(url, ssl=False) as resp:
            if resp.status == 200:
                html_code = await resp.text()
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
```

输出：

```text
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
```

从上面的输出可以看出，网站首页标题的输出顺序跟它们的 URL 在列表中的顺序没有关系。代码的第11行到第13行创建了`ClientSession`对象，通过它的`get`方法可以向指定的 URL 发起请求，如第14行所示，跟`requests`中的`Session`对象并没有本质区别，唯一的区别是这里使用了异步上下文。代码第16行的`await`会让因为 I/O 操作阻塞的子程序放弃对 CPU 的占用，这使得其他的子程序可以运转起来去抓取页面。代码的第17行和第18行使用了正则表达式捕获组操作解析网页标题。`fetch_page_title`是一个被`async`关键字修饰的异步函数，调用该函数会获得协程对象，如代码第35行所示。后面的代码跟之前的例子没有什么区别，相信大家能够理解。

### 单线程版本

通过上面的 URL 下载“美女”频道共`90`张图片。

```python
"""
example04.py - 单线程版本爬虫
"""
import os

import requests


def download_picture(url):
    filename = url[url.rfind('/') + 1:]
    resp = requests.get(url)
    if resp.status_code == 200:
        with open(f'images/beauty/{filename}', 'wb') as file:
            file.write(resp.content)


def main():
    if not os.path.exists('images/beauty'):
        os.makedirs('images/beauty')
    for page in range(3):
        resp = requests.get(f'https://image.so.com/zjl?ch=beauty&sn={page * 30}')
        if resp.status_code == 200:
            pic_dict_list = resp.json()['list']
            for pic_dict in pic_dict_list:
                download_picture(pic_dict['qhimg_url'])

if __name__ == '__main__':
    main()
```

在 macOS 或 Linux 系统上，我们可以使用`time`命令来了解上面代码的执行时间以及 CPU 的利用率，如下所示。

```bash
time python3 example04.py
```

下面是单线程爬虫代码在我的电脑上执行的结果。

```text
python3 example04.py  2.36s user 0.39s system 12% cpu 21.578 total
```

这里我们只需要关注代码的总耗时为`21.578`秒，CPU 利用率为`12%`。

### 多线程版本

我们使用之前讲到过的线程池技术，将上面的代码修改为多线程版本。

```python
"""
example05.py - 多线程版本爬虫
"""
import os
from concurrent.futures import ThreadPoolExecutor

import requests


def download_picture(url):
    filename = url[url.rfind('/') + 1:]
    resp = requests.get(url)
    if resp.status_code == 200:
        with open(f'images/beauty/{filename}', 'wb') as file:
            file.write(resp.content)


def main():
    if not os.path.exists('images/beauty'):
        os.makedirs('images/beauty')
    with ThreadPoolExecutor(max_workers=16) as pool:
        for page in range(3):
            resp = requests.get(f'https://image.so.com/zjl?ch=beauty&sn={page * 30}')
            if resp.status_code == 200:
                pic_dict_list = resp.json()['list']
                for pic_dict in pic_dict_list:
                    pool.submit(download_picture, pic_dict['qhimg_url'])


if __name__ == '__main__':
    main()
```

执行如下所示的命令。

```bash
time python3 example05.py
```

代码的执行结果如下所示：

```text
python3 example05.py  2.65s user 0.40s system 95% cpu 3.193 total
```

### 异步I/O版本

我们使用`aiohttp`将上面的代码修改为异步 I/O 的版本。为了以异步 I/O 的方式实现网络资源的获取和写文件操作，我们首先得安装三方库`aiohttp`和`aiofile`，命令如下所示。

```bash
pip install aiohttp aiofile
```

`aiohttp` 的用法在之前的课程中已经做过简要介绍，`aiofile`模块中的`async_open`函数跟 Python 内置函数`open`的用法大致相同，只不过它支持异步操作。下面是异步 I/O 版本的爬虫代码。

```python
"""
example06.py - 异步I/O版本爬虫
"""
import asyncio
import json
import os

import aiofile
import aiohttp


async def download_picture(session, url):
    filename = url[url.rfind('/') + 1:]
    async with session.get(url, ssl=False) as resp:
        if resp.status == 200:
            data = await resp.read()
            async with aiofile.async_open(f'images/beauty/{filename}', 'wb') as file:
                await file.write(data)


async def fetch_json():
    async with aiohttp.ClientSession() as session:
        for page in range(3):
            async with session.get(
                url=f'https://image.so.com/zjl?ch=beauty&sn={page * 30}',
                ssl=False
            ) as resp:
                if resp.status == 200:
                    json_str = await resp.text()
                    result = json.loads(json_str)
                    for pic_dict in result['list']:
                        await download_picture(session, pic_dict['qhimg_url'])


def main():
    if not os.path.exists('images/beauty'):
        os.makedirs('images/beauty')
    loop = asyncio.get_event_loop()
    loop.run_until_complete(fetch_json())
    loop.close()


if __name__ == '__main__':
    main()
```

执行如下所示的命令。

```bash
time python3 example06.py
```

代码的执行结果如下所示：

```text
python3 example06.py  0.82s user 0.21s system 27% cpu 3.782 total
```

### 总结

通过上面三段代码执行结果的比较，我们可以得出一个结论，使用多线程和异步 I/O 都可以改善爬虫程序的性能，因为我们不用将时间浪费在因 I/O 操作造成的等待和阻塞上，而`time`命令的执行结果也告诉我们，单线程的代码 CPU 利用率仅仅只有`12%`，而多线程版本的 CPU 利用率则高达`95%`；单线程版本的爬虫执行时间约`21`秒，而多线程和异步 I/O 的版本仅执行了`3`秒钟，后者略多一点。另外，在运行时间差别不大的情况下，多线程的代码比异步 I/O 的代码耗费了更多的 CPU 资源，这是因为多线程的调度和切换也需要花费 CPU 时间。至此，三种方式在 I/O 密集型任务上的优劣已经一目了然，当然这只是在我的电脑上跑出来的结果。如果网络状况不是很理想或者目标网站响应很慢，那么使用多线程和异步 I/O 的优势将更为明显，有兴趣的读者可以自行试验。

