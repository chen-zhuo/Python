# Web后端开发

## Web开发框架

**Web开发框架存在的意义就在于可以快速便捷的构建应用，而不用去在意那些没必要的技术细节(协议、报文、数据结构)。**

在今天，Python里有很多Web开发框架用来帮助你轻松创建Web应用。到2020年为止，基于Python创建的的web应用已经非常多了，国外知名的有youtube.com、instagram、reditt，国内有知乎、豆瓣等等。这些网站分别用到了不同的web框架来实现的，我们今天会一一讲到。

### Django简介

![django](image/django.jpg)

**Django：Python最知名、最有代表性的Web应用开发框架。**

Django应该是最出名的Python框架，GAE甚至Erlang都有框架受它影响。

Django是走大而全的方向，它最出名的是其全自动化的管理后台：只需要使用起ORM，做简单的对象定义，它就能自动生成数据库结构、以及全功能的管理后台。

**Django遵循了MVC开发模式，并将这个模式命名为MTV(MTV模式是Python中独有的)：M Model(数据模型，用于后端数据库模型定义和处理模块)，T Templates(模版，用于前端显示信息)，V View(视图，用于接收客户端请求、处理Model、渲染返回信息给客户端等)。**

Django优势劣势：

```
Django优势：
1.一站式开发解决方案，拧包入住
2.各种组件集成高度成熟，配置齐全
3.用户模型、权限认证体系健全
4.ORM数据库管理功能简单方便
5.自带后台管理功能

Django劣势：
1.配置相对复杂
2.简单应用采用Django有一种杀鸡用牛刀的感觉
```
### Flask简介

![flask](image/flask.png)

**Flask：一个基于Werkzeug WSGI工具箱和Jinja2模板引擎的轻量级Web应用框架。**

Flask也被称为“microframework”，因为它使用简单的核心，用extension增加其他功能，这也让框架相对比较灵活。核心思想是Flask只完成基本的功能，别的功能都是靠各种第三方插件来完成的，实现了模块高度化定制。

**如果说Django是大而全的方案代表，那么Flask就是小而精的方案代表。**

Flask的两个主要核心应用是Werkzeug和模板引擎Jinja，除此以外都是可以自由组装的。常用的Flask插件如下：

```
Flask-SQLalchemy：操作数据库;
Flask-migrate：管理迁移数据库;
Flask-Mail:邮件;
Flask-WTF：表单;
Flask-script：插入脚本;
Flask-Login：认证用户状态;
Flask-RESTful：开发REST API的工具;
Flask-Bootstrap：集成前端Twitter Bootstrap框架;
Flask-Moment：本地化日期和时间;
```
Flask优势劣势：

```
Flask优势：
1.项目结构和配置简单
2.组件可以自由拆装
3.小项目或临时性项目比较适用

Flask劣势：
1.高度自定义组件带来的就是组件之间的兼容性问题，严重不适合大型应用，例如蓝图(blueprint)机制跟Django的url配置比起来其实差得很远
```

### Tornado简介

![tornado](image/tornado.png)

**Tornado：异步非阻塞IO的Python Web框架。**

Tornado的全称是Torado Web Server，严格意义上来说Tornado不是一个Web框架，而是一个基于Python实现的异步处理框架，只是自带了WSGI处理相关的功能。

**Tornado和Flask一样，除了基本的Web处理功能和模版之外，其他功能组件都需要自行拼装。**

Tornado优势劣势：

```
Tornado优势：
1.短小精悍，性能比较好，不依赖Python多进程/多线程
2.支持异步非阻塞IO处理方式
3.支持websocket

Tornado劣势：
1.过于精简，只适用于纯接口化服务或者小型网站应用
```
## Web环境

一般在进行Web项目开发时有两种环境：`development（开发环境）`、`production（生产环境）`。

### 开发环境

`development（开发环境）`：**即线下开发Web项目的环境，因为开发过程中总会遇到报错、响应不正常等各种情况，因此需要开发环境来试错调试。同时，在这种环境中要尽量暴露Web项目中的错误，方便于开发人员修改，提供更好、更高质量的Web项目。**

### 生产环境

`production（生产环境）`：**即线上部署Web项目的环境，因为最终我们开发的Web项目都要部署到线上去，因此需要生产环境来测试。同时，在这种环境中要尽量隐藏Web项目中代码的错误，让用户的到一个好的体验，同时也一定程度上防范骇客通过错误漏洞对网站发起攻击。**

## 总结

今天给大家介绍了各种Python的Web开发框架和Web开发环境，因为流行度和易用性的关系，常用的框架主要包括Django、Flask、Tornado。针对于最常用的三种框架，我给大家一个使用建议：

**Django：适用于正式项目、大型项目，确定需要长期开发和维护的项目。**

**Flask：适用于小型项目、临时性的、需求简单的项目。**

**Tornado：小型项目、临时性项目或者一些简单的接口服务。因为Tornado天生支持异步，所以很多需要做异步IO服务的也可以选择Tornado。**

**开发环境：在开发Web项目时使用的环境。**

**生产环境：在部署Web项目时使用的环境。**

