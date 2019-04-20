# PyCharm开发工具

PyCharm是一个使用Python语言开发时提高其效率的工具

### 项目文件夹

使用PyCharm打开一个文件夹时，会在这个文件夹下面生成了一个名称为 .idea 文件夹，PyCharm通过 .idea 文件夹来标记这个文件夹成为项目文件夹。若不想成为项目文件夹，删除.idea文件夹即可。

**提醒：使用PyCharm查看文件夹列表时，项目文件夹会带有一个小黑块。**

### Terminal

PyCharm下面的Terminal作用类似于命令行窗口，可以执行相应的命令。

### 安装包命令

##### 指令下载安装

```
pip install XXX(包名)
```

##### 网址下载安装

从 https://pypi.douban.com/simple 这个网址下载对应的包

```
pip install XXX(包名) -i https://pypi.douban.com/simple
```

##### 文件信息下载安装

根据requirements.txt文件里面的安装包的包名和版本号信息，来下载对应的包

```
# requirements.txt文件

selenium==3.6.0
Scrapy==1.5.1
pymongo==3.5.1
Pillow==5.2.0
lxml==4.2.5
```

```
pip install -r requirements.txt
```

### Pycharm常用快捷键

```
ctrl+/  ---  单行注释（注释掉当前行内容）

ctrl+d  ---  复制粘贴（复制当前行代码并粘贴下面）

ctrl+y  ---  删除（删除当前行代码）
```


