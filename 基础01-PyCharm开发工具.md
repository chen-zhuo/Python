# *PyCharm* 开发工具

### *PyCharm*

*PyCharm* 是一个使用 `Python` 作为开发语言时提高效率的工具。

##### *PyCharm* 快捷键

```
ctrl+/  ---  注释（注释掉当前行或选择的内容）

ctrl+d  ---  复制粘贴（复制当前行代码并粘贴下面）

ctrl+y  ---  删除（删除当前行代码）
```

##### 项目文件夹

使用 *PyCharm* 打开一个普通文件夹时，会在这个文件夹下面生成了一个名称为 `.idea` 文件夹，*PyCharm* 通过该文件夹来标记所在文件夹为项目文件夹。若不想成为项目文件夹，删除 `.idea` 文件夹即可。

**提醒：使用PyCharm查看文件夹列表时，项目文件夹会带有一个小黑块。**

![QQ截图20191202234418](image/QQ截图20191202234418.png)

##### 个性化配色

通过下面6个步骤来进行配色：

1. *File --- Settings --- Apperance & Behavior --- Appearance --- Theme: Darcula*
   ![1354564-20180403215540009-1958749858](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403215540009-1958749858.png)
2. *File --- Settings --- Editor --- Color Scheme Font --- Scheme: Monokai*
   ![1354564-20180403215733932-609955603](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403215733932-609955603.png)
3. 选择 *Monokai* 后，点击右边的“齿轮”，选择 *Duplicate*，备份原 *Monokai* 方案。
   ![1354564-20180403220136728-1625767838](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403220136728-1625767838.png)
4. 再选择新创建的 *Monokai Copy*。![1354564-20180403220314140-327909293](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403220314140-327909293.png)
5. *File --- Settings --- Editor --- Color Scheme --- Python --- Scheme: Monokai copy*
   ![1354564-20180403220457059-2139653139](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403220457059-2139653139.png)
6. *File --- Settings --- Editor --- Color Scheme Font --- Use color scheme font*（勾选）
   ![1354564-20180403220941608-1669928136](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403220941608-1669928136.png)

完成配色后的代码：
![1354564-20180403215217533-697643815](../../../%E9%85%8C/Desktop/Python/image/1354564-20180403215217533-697643815.png)

### *Run*

*PyCharm* 下面的 *Run* 窗口就是程序运行结果的展示窗口。

![QQ截图20191203225101](image/QQ截图20191203225101.png)

### *Terminal*

##### *Terminal* 简介

*PyCharm* 下面的 *Terminal* 窗口就相当 *CMD* 命令行窗口，执行相应的命令。

![QQ截图20191203223918](image/QQ截图20191203223918.png)

##### 第三方库安装命令

**第三方库**：网络上公共的开源库。

**第三库是先下载才能使用的，所以安装命令是针对第三方库的，所有的命令都在 *Terminal* 或 *cmd* 中执行**

```python
# 指令安装（从默认的pip源中下载，速度较慢）
pip install XXX(包名)

# 豆瓣镜像安装（从国内镜像源网站中下载，速度快）
pip install XXX(包名) -i http://pypi.douban.com/simple
# 豆瓣镜像安装（并添加信任地址）
pip install XXX(包名) -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
# 阿里云镜像源安装（从国内镜像源网站中下载，速度快）
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