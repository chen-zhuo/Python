# Flask数据库

大部分程序都需要保存数据，所以不可避免要使用数据库。用来操作数据库的数据库管理系统（DBMS）有很多选择，对于不同类型的程序，不同的使用场景，会有不同的选择。

### ORM

**对象关系映射（简称ORM），是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换**。从效果上说，它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”，**简单的说：ORM相当于中继数据。**

##### SQLAlchemy

**SQLAlchemy 是 Python 语言下的一款开源软件，提供了 SQL 工具包及对象关系映射（ORM）工具。**通过定义 Python 类来表示数据库里的一张表（类属性表示表中的字段 / 列），这个类称为**模型类**，类中的属性称为**字段**，通过对这个类进行各种操作来代替写 SQL 语句。

Flask 有大量的第三方扩展，这些扩展可以简化和第三方库的集成工作。我们下面将使用一个叫做 [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.3/) 的官方扩展来集成 SQLAlchemy。

首先安装它：

```
pip install flask-sqlalchemy
```

大部分扩展都需要执行一个“初始化”操作。你需要导入扩展类，实例化并传入 Flask 程序实例：

```python
from flask_sqlalchemy import SQLAlchemy  # 导入扩展类
app = Flask(__name__)
db = SQLAlchemy(app)  # 初始化扩展，传入程序实例 app
```

##### 配置连接

Flask 提供了一个统一的接口来写入和获取这些配置变量：`Flask.config` 字典。配置变量的名称必须使用大写，写入配置的语句一般会放到扩展类实例化语句之前。

下面写入了一个 `SQLALCHEMY_DATABASE_URI` 变量来告诉 SQLAlchemy 数据库连接地址：

```python
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy  # 导入扩展类

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = "mysql+pymysql://用户:密码@IP地址:端口/数据库?charset=utf8"  # 连接MySQL数据库
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # 关闭对模型修改的监控
db = SQLAlchemy(app)  # 在扩展类实例化前加载配置
```

### CRUD

**CRUD是指在做计算处理时的增加(Create)、检索(Retrieve)、更新(Update)和删除(Delete)几个单词的首字母简写，主要被用在描述软件系统中数据库或者持久层的基本操作功能。**

##### 建立模型

目前我们有两类数据要保存：用户信息和电影条目信息。我们新建一个 `mysql_model.py` 文件，创建两个模型类来表示这两张表：

```python
from app import db

# 模型类要声明继承db.Model
# 类属性（字段）要实例化db.Column，传入字段类型，可以添加额外参数设置字段
class User(db.Model):
    __tablename__ = "user"  # 表名将会是user
    id = db.Column(db.Integer, primary_key=True)  # 主键
    name = db.Column(db.String(20))  # 名字

class Movie(db.Model):
    __tablename__ = "movie"  # 表名将会是movie
    id = db.Column(db.Integer, primary_key=True)  # 主键
    title = db.Column(db.String(60))  # 电影标题
    year = db.Column(db.String(4))  # 电影年份
```

类属性基本的格式：`变量 = db.Column(字段类型, 额外参数)`

| 字段类           | 说明                                          |
| ---------------- | --------------------------------------------- |
| db.Integer       | 整型                                          |
| db.String (size) | 字符串，size 为最大长度，比如 `db.String(20)` |
| db.Text          | 长文本                                        |
| db.DateTime      | 时间日期，Python `datetime` 对象              |
| db.Float         | 浮点数                                        |
| db.Boolean       | 布尔值                                        |

| 额外参数    | 接收值 | 作用                 |
| ----------- | ------ | -------------------- |
| primary_key | 布尔值 | 设置该字段是否为主键 |
| nullable    | 布尔值 | 是否允许为空值       |
| index       | 布尔值 | 是否设置索引         |
| unique      | 布尔值 | 是否允许重复值       |
| default     | 自定义 | 根据字段类设置默认值 |

##### 建立数据表

所有的CRUD操作都是建立在数据表上的，现在我们建立好了模型，就可以通过模型来建立数据表。两个模型User、Movie就对应两个数据表user、movie：

```python
from hello_flask import db

class User(db.Model):  # 表名将会是 user
    id = db.Column(db.Integer, primary_key=True)  # 主键
    name = db.Column(db.String(20))  # 名字

class Movie(db.Model):  # 表名将会是 movie
    id = db.Column(db.Integer, primary_key=True)  # 主键
    title = db.Column(db.String(60))  # 电影标题
    year = db.Column(db.String(4))  # 电影年份

def start():
    db.create_all()

if __name__ == "__main__":
    try:
        start()
        print("create table successful.")
    except:
        print("create table failed !!!")



