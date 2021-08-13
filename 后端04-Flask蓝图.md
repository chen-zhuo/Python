# Flask蓝图

项目开发是一个非常耗时间和精力的工程，如果我们将所有的Flask请求方法都写在 `manage.py` 文件下的话，这会让我们后期管理和维护代码变得复杂且困难，因此我们需要对程序进行模块化的处理。

### 蓝图 Blueprint

##### Blueprint简介

**Blueprint 是一个存储视图方法的容器，这些操作在这个Blueprint 被注册到一个应用之后就可以被调用，Flask 可以通过Blueprint来组织URL以及处理请求。**

Flask使用Blueprint让应用实现模块化，在Flask中，Blueprint具有如下属性：

- 一个项目可以具有多个Blueprint
- 可以将一个Blueprint注册到任何一个未使用的URL下比如 “/”、“/sample”或者子域名
- 在一个应用中，一个模块可以注册多次
- Blueprint可以单独具有自己的模板、静态文件或者其它的通用操作方法，它并不是必须要实现应用的视图和函数的
- 在一个应用初始化时，就应该要注册需要使用的Blueprint

**但是一个Blueprint并不是一个完整的应用，它不能独立于应用运行，而必须要注册到某一个应用中。**

