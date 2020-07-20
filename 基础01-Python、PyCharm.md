

##### Python内存管理

Python 的内存管理是由解释器负责的，有三大机制：**对象的引用计数、垃圾回收、内存池**。

**对象引用计数**：Python 内部使用引用计数，来追踪内存中的对象，所有对象都有引用计数。

引用计数增加：

1. 一个对象分配一个新名称
2. 将其放入一个容器中（如列表、元组或字典）

引用计数减少：

1. 使用 `del` 语句对对象别名显示的销毁
2. 引用超出作用域或被重新赋值

**垃圾回收**：当一个对象的引用计数归零时，它将被垃圾收集机制处理掉。

**内存池**：为了加速Python的执行效率，Python提供了对内存的垃圾收集机制，将不用的内存放到内存池而不是返回给操作系统，用于管理对小块内存的申请和释放。

##### Python 文件

Python 文件是以 `.py` 为后缀名的，一个 `.py` 文件就是一个模块。

库：具有相关功能模块的集合。

库也是 Python 的特色，即具有强大的标准库、第三方库以及自定义模块

**标准库**

标准库：安装 Python 的时候默认自带的库。

常见的有：`os` 系统操作，`time` 时间，`random` 随机，`re` 正则，`logging` 日志，`queue` 队列，`threading` 线程，`multiprocessing` 进程

**第三方库**

第三方库：第三方机构发布的特定功能的模块，需要下载安装后才能使用的库。

常见的有：`lxml` 网页解析库，`scrapy` 爬虫框架，`django` 网站框架，`flask` 网站框架，`virtualenv` 虚拟环境

**自定义模块**

自定义模块：用户自行编写的模块。

### *PyCharm*

*PyCharm* 是一个使用 Python 作为开发语言时提高效率的工具，可以在[Pycharm官网](https://www.jetbrains.com/pycharm/download/#section=windows)中下载。

##### 永久激活

*Pycharm* 分为社区版和专业版，社区版是免费使用的，专业版是限时免费使用，后期需要激活的才能使用的，若需要永久激活参看 [Pycharm永久激活](https://www.cnblogs.com/jiyu-hlzy/p/11726732.html)。

##### 背景颜色

File --- Settings(设置) --- Editor(编辑器) --- Color Scheme(颜色方案) --- Scheme: Default(默认白色)、Github(白色)、Darcula(深蓝色)、Monokai(**推荐黑色**)

![QQ截图20200203163215](image/QQ截图20200203163215.png)

![QQ截图20200203163605](image/QQ截图20200203163605.png)

##### 字体设置

File --- Settings(设置) --- Editor(编辑器) --- Font(字体) --- Font: ...(**字体样式自定义**) --- Size: 18(**推荐字体大小**) --- Line spacing: 1.0(**推荐行间距**)

![QQ截图20200203164426](image/QQ截图20200203164426.png)

##### 个性化配色

通过下面6个步骤来进行配色：

1. *File --- Settings --- Apperance & Behavior --- Appearance --- Theme: Darcula*
   ![1354564-20180403215540009-1958749858](image/1354564-20180403215540009-1958749858.png)
2. *File --- Settings --- Editor --- Color Scheme Font --- Scheme: Monokai*
   ![1354564-20180403215733932-609955603](image/1354564-20180403215733932-609955603.png)
3. 选择 *Monokai* 后，点击右边的“齿轮”，选择 *Duplicate*，备份原 *Monokai* 方案。
   ![1354564-20180403220136728-1625767838](image/1354564-20180403220136728-1625767838.png)
4. 再选择新创建的 *Monokai Copy*。![1354564-20180403220314140-327909293](image/1354564-20180403220314140-327909293.png)
5. *File --- Settings --- Editor --- Color Scheme --- Python --- Scheme: Monokai copy*
   ![1354564-20180403220457059-2139653139](image/1354564-20180403220457059-2139653139.png)
6. *File --- Settings --- Editor --- Color Scheme Font --- Use color scheme font*（勾选）
   ![1354564-20180403220941608-1669928136](image/1354564-20180403220941608-1669928136.png)

完成配色后的代码：
![1354564-20180403215217533-697643815](image/1354564-20180403215217533-697643815.png)

##### *PyCharm* 快捷键

下面快捷键只针对于PyCharm开发工具有效：

```
Ctrl+/  ---  注释（注释掉当前行或选择的内容）

Ctrl+D  ---  复制粘贴（复制当前行代码并粘贴下面）

Ctrl+Y  ---  删除（删除当前行代码）

Ctrl+R  ---  替换
```

##### 项目文件夹

使用 *PyCharm* 打开一个普通文件夹时，会在这个文件夹下面生成了一个名称为 `.idea` 文件夹，*PyCharm* 通过该文件夹来标记所在文件夹为项目文件夹。若不想成为项目文件夹，删除 `.idea` 文件夹即可。

**提醒：使用PyCharm查看文件夹列表时，项目文件夹会带有一个小黑块。**

![QQ截图20191202234418](image/QQ截图20191202234418.png)

##### *Run*

*PyCharm* 下面的 *Run* 窗口就是程序运行结果的展示窗口。

![QQ截图20191203225101](image/QQ截图20191203225101.png)

##### *Terminal*

*PyCharm* 下面的 *Terminal* 窗口就相当 *CMD* 命令行窗口，执行相应的命令。

![QQ截图20191203223918](image/QQ截图20191203223918.png)

##### 第三方库安装命令

**第三方库**：网络上公共的开源库。

**第三库是先下载才能使用的，所以安装命令是针对第三方库的，所有的命令都在 *Terminal* 或 *cmd* 中执行**

```python
# 指令安装（从默认的pip源中下载，速度较慢）
pip install XXX(包名)

# 下面的命令都是从国内镜像源网站中下载，下载速度快
# 清华大学镜像源安装（三方库齐全，推荐）
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple XXX(包名)
# 豆瓣镜像源安装
pip install XXX(包名) -i http://pypi.douban.com/simple
# 阿里云镜像源安装
pip install XXX(包名) -i http://mirrors.aliyun.com/pypi/simple/

# 文件信息下载安装
# requirements.txt文件：
'''
selenium==3.6.0
Scrapy==1.5.1
pymongo==3.5.1
lxml==4.2.5
'''
# 根据requirements.txt文件里面的安装库的包名和版本号信息，来下载对应的包
pip install -r requirements.txt

#列出所有已安装的三方库
pip list   

# 列出当前已安装且过期的第三方库
pip list --outdated  

# 更新已安装的三方库
pip install --upgrade 库名  
```

