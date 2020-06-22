# Scrapy框架

### Scrapy简介

**Scrapy 是一个基于 Twisted 的多线程异步处理框架，所以scrapy是自带多线程的。**

优点：架构清晰， 模块之间的耦合程度低（相互影响小），可扩展性极强，可以灵活完成各种需求，我们只需要定制开发几个模块就可以轻实现一个爬虫。

**Scrapy数据流**

![img](https://upload-images.jianshu.io/upload_images/1370011-a3be891e6a48dad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/706/format/webp)

1、引擎打开起点网站(open a domain)，向处理该网站的Spider请求第⼀个要爬取的URL。

2、引擎从Spider中获取到第⼀个要爬取的URL并在调度器(Scheduler)调度。

3、引擎向调度器请求下⼀个要爬取的URL。

4、调度器返回下⼀个要爬取的URL给引擎，引擎将URL通过下载中间件转发给下载器(Downloader)。

5、页面下载完毕，下载器生成页面的Response，并将其通过下载中间件发送给引擎。

6、引擎从下载器中接收到Response并通过Spider中间件发送给Spider处理。

7、Spider在Response中爬取到的Item和新的Request给引擎。

8、引擎将爬取到的Item给Item Pipeline，将Request给调度器。

9、(从第⼆步)重复直到调度器中没有更多地request，引擎关闭该⽹站。

### Scrapy安装

##### Twisted安装

在安装Scrapy时，可能会报一个关于Twisted的错误，这是因为scrapy底层使用twisted框架，因此我们需要先安装Twisted。

```
Twisted-18.9.0-cp36-cp36m-win_amd64

18.9.0：版本号

cp36：python3.6

win_amd64：windows64位
```

根据本地情况下载合适的Twisted，放到命令行要执行的位置，执行下面命令：

```
pip install Twisted-18.9.0-cp36-cp36m-win_amd64
```

##### Scrapy安装

执行下面命令安装Scrapy

```
pip install scrapy
```

另外，**Scrapy也是异步的，是基于Twisted事件驱动的**。在任何情况下，都不要写阻塞的代码。例如：

1. 访问文件、数据库或者Web

2. 产生新的进程并需要处理新进程的输出，如运行shell命令

3. 执行系统层次操作的代码，如等待系统队列