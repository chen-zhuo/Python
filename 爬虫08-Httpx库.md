# Httpx库

前面我们学习了requests库，这是写爬虫最简单、最易用的第三方库，虽然使用requests库可以爬取绝大部分网站，但它也不是万能的，它也有自己的局限，例如：不支持HTTP2.0、只能同步请求（也就是第一个请求返回响应后才会去进行第二个请求）效率不高。

因此我们有必要多掌握一个写爬虫的库，毕竟没有万能的工具，那就只有成为会多种工具的人，让自己万能。接下来要介绍的第三库就是：Httpx库。

Httpx库官方文档：https://www.python-httpx.org/

## 初识Httpx库

HTTPX是Python3的一个功能齐全的HTTP第三方库，它提供同步和异步API，并支持HTTP/1.1和HTTP/2。

### 安装Httpx

安装Httpx命令：

```
pip install httpx
```

