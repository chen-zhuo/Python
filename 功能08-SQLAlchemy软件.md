# SQLAlchemy软件

## 概述

### 对象关系映射

面向对象是从软件工程基本原则（如耦合、聚合、封装）的基础上发展起来的，而关系数据库则是从数学理论发展而来的，两套理论存在显著的区别。为了解决这个不匹配的现象，对象关系映射技术应运而生。

**对象关系映射（英语：Object Relational Mapping，简称ORM），是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换，它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”。简单的说：ORM相当于中继数据。**

### SQLAlchemy简介

**SQLAlchemy是Python编程语言下的一款开源软件，提供了SQL工具包及对象关系映射（ORM）工具，使用MIT许可证发行，为高效和高性能的数据库访问设计，实现了完整的企业级持久模型。**其理念是，SQL数据库的量级和性能重要于对象集合；而对象集合的抽象又重要于表和行。

SQLAlchemy与数据库关系图如下：

![938876-20160814190600531-930266102](image/938876-20160814190600531-930266102.png)

SQLAlChemy的三个最重要的部分是：**SQL表达式语言、对象关系映射器(ORM)以及Core 。**

**SQL表达式语言是一个完全独立于ORM包的工具包，它提供了一个构造由可组合对象表示的SQL表达式的系统，针对特定事务范围内的目标数据库“执行”这些SQL表达式，并返回结果集。**

使用Core和SQL表达式语言提供了一个以模式为中心的数据库视图，以及一个面向不变性的编程范例，而ORM则在此基础上构建了一个以域为中心的数据库视图，提供使用映射到数据库模式的域对象模型的方法，具有更明显的面向对象和依赖于可变性的编程范例。

**由于关系数据库本身是可变服务，不同之处在于Core/SQL表达式语言是面向命令的，而ORM是面向状态的。**在使用ORM时，SQL语句的构造方式与使用Core时基本相同，**但是插入、更新和删除(即 DML)的任务(这里指的是数据库中业务对象的持久性)是使用称为 unit of work ，它将针对可变对象的状态更改转换为INSERT、UPDATE和DELETE构造，然后根据这些对象调用这些构造。**

## 简单使用

本章节使用的SQLAlchemy版本为1.4（安装要求最低python3.6），所有的例子也都适用于1.4以上的版本，有一点要注意的就是1.4版本与1.3版本有较大改动，以下案例或许不能通用。

### 安装检查

下载安装SQLAlchemy：

```
pip install SQLAlchemy
```

检查SQLAlchemy的安装版本：

```python
import sqlalchemy

print(sqlalchemy.__version__)  # 1.4.29
```

### 建立连接

**任何SQLAlchemy程序开始都是有一个只为特定数据库服务器创建一次的名为 `Engine` 的全局对象，此对象会通过一个URL字符串（该字符串将描述如何连接到数据库主机或后端）进行配置连接到特定数据库的中心源。**

```python
from sqlalchemy import create_engine

# 初始化数据库连接
engine = create_engine("mysql+pymysql://用户名:密码@IP:端口/数据库名称", 
         echo=True,  # 输出SQL执行过程，调试的时候更方便
         future=True  # 启用2.0版API（用于1.4及以上版本）
         max_overflow=10,  # 最大连接数（默认为5）
         pool_size=10,  # 连接池大小
         pool_timeout=60,  # 池中没有线程最多等待的时间，否则报错
         pool_recycle=60 * 60 * 2  # 设置时间以限制数据库多久没连接自动断开，它默认为-1或者没有超时
        )

# Engine对象的唯一目的提供到数据库的连接单元，称为Connection。
conn = engine.connect()
# 如果希望将Engine对象的使用范围限制在特定上下文中，最好使用Python上下文管理器表单。
with engine.connect() as conn:
    ...
```

### 执行文本SQL

`text()` 方法允许编写文本SQL语句。

`conn.execute()` 方法执行文本SQL语句。

```python
from sqlalchemy import text
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")

# 通过conn.execute()方法执行文本SQL语句
with engine.connect() as conn:
    result = conn.execute(text("select 'hello world'"))
    print(result.all())  # [('hello world',)]
    result = conn.execute(text("SELECT 3*8"))
    print(result.all())  # [(24,)]

# 使用冒号格式“变量名:变量”接受变量传递的参数
with engine.connect() as conn:
    result = conn.execute(text("SELECT 3*8 WHERE 5>:y"), {"y": 1})
    print(result.all())  # [(24,)]
    result = conn.execute(text("SELECT 3*8 WHERE 5>:y"), {"y": 10})
    print(result.all())  # []

# 同样的我们还可以对表做出一些更改操作
with engine.connect() as conn:
    # 建表语句
    conn.execute(text("CREATE TABLE some_table (x int, y int)"))
    # 插入语句
    conn.execute(text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),[{"x": 1, "y": 1}, {"x": 2, "y": 4}, {"x": 6, "y": 8}, {"x": 9, "y": 10}])
```

!> 注意：SQLAlchemy的1.4.29版本已经没有 `conn.commit()` 方法来提交更改，都是 `conn.execute()` 边走边执行。

### 结果遍历

`Result` 有很多用于获取和转换行的方法，例如上面 `Result.all()` 方法一次性接受所有结果，它返回所有 Row 物体。它还实现了Python迭代器接口，以便我们可以迭代 Row 直接对象。这个 `Row` 对象本身的作用类似于Python named tuples. 下面我们将演示访问行的各种方法。

- **元组赋值** -这是Python最惯用的风格，即在接收到变量时将变量按位置分配给每一行：

  ```python
  with engine.connect() as conn:
      result = conn.execute(text("select x, y from some_table"))
      for x, y in result:
          print(f'x: {x}, y: {y}')
  ```

- **整数索引** -元组是Python序列，因此也可以进行常规整数访问：

  ```python
  with engine.connect() as conn:
      result = conn.execute(text("select x, y from some_table"))
          for row in result:
              x = row[0]
  ```

- **属性名称** -由于这些是Python命名的元组，这些元组具有与每个列的名称相匹配的动态属性名。这些名称通常是SQL语句为每行中的列指定的名称。虽然它们通常是相当可预测的，也可以由标签控制，但在定义较少的情况下，它们可能会受到特定于数据库的行为的影响：

  ```python
  with engine.connect() as conn:
      result = conn.execute(text("SELECT x, y FROM some_table"))
      for row in result:
          print(f"x: {row.x}  y: {row.y}")
  '''
  x: 1  y: 1
  x: 2  y: 4
  x: 6  y: 8
  x: 9  y: 10
  '''
  ```

- **映射访问** -以Python形式接收行 **映射** 对象，它本质上是Python的公共接口的只读版本 `dict` 对象 `Result` 可能是 **转化** 变成一个 `MappingResult` 对象使用 `Result.mappings()` 修饰符；这是一个结果对象，它生成类似dictionary的 `RowMapping` 对象而不是 `Row` 物体：：

  ```python
  with engine.connect() as conn:
      result = conn.execute(text("select x, y from some_table"))
      for dict_row in result.mappings():
          x = dict_row['x']
          y = dict_row['y']
  ```

## 常用类

### Table类

现在我们来学习如何使用ORM来操作数据库，**首先需要创建基本结构 Table 表对象来对应操作的数据库表。**在SQLAlchemy中提供了Table类来创建表对象，构造函数如下：

```python
from sqlalchemy import Table

Table.__init__(self, name, metadata, *args, **kwargs)
```

| 名称            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| **name**        | **数据库表名**                                               |
| **metadata**    | **共享的元数据**                                             |
| **\*args**      | **Column，用来定义列**                                       |
| **\*\*kwargs**  | **数据库表属性的可变参数（具体如下）**                       |
| schema          | 表的结构名称，默认None                                       |
| autoload        | 自动从现有表中读入表结构，默认False                          |
| autoload_with   | 从其他engine读取结构，默认None                               |
| include_columns | 如果autoload设置为True，则此项数组中的列明将被引用，没有写的列明将被忽略，None表示所有都列明都引用，默认None |
| mustexist       | 如果为True，表示这个表必须在其他的python应用中定义，必须是metadata的一部分，默认False |
| useexisting     | 如果为True，表示这个表必须被其他应用定义过，将忽略结构定义，默认False |
| owner           | 表所有者，用于Orcal，默认None                                |
| quote           | 设置为True，如果表明是SQL关键字，将强制转义，默认False       |
| quote_schema    | 设置为True，如果列明是SQL关键字，将强制转义，默认False       |
| mysql_engine    | mysql专用，可以设置'InnoDB'或'MyISAM'                        |

### Column类

**在上面的Table类中有一参数 `*args` 是专门用来定义列的，及数据库表中的字段。**同样的，在SQLAlchemy中提供了Column类来创建表对象，构造函数如下：

```python
from sqlalchemy import Column

Column.__init__(self, name, type_, *args, **kwargs)
```

| 名称             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| **name**         | **列名**                                                     |
| **type_**        | **列的类型，详见 `sqlalchemy.types`**                        |
| **\*args**       | **列的约束参数（具体如下）**                                 |
| ForeignKey('列') | 列的外键                                                     |
| ColumnDefault    | 默认                                                         |
| Sequenceobjects  | 序列                                                         |
| key              | 列的别名                                                     |
| **\*\*kwargs**   | **列中的可变参数（具体如下）**                               |
| primary_key      | 如果为True，则是主键                                         |
| nullable         | 是否可为Null，默认是True                                     |
| default          | 默认值，默认是None                                           |
| index            | 是否是索引，默认是True                                       |
| unique           | 是否唯一键，默认是False                                      |
| onupdate         | 指定一个更新时候的值，这个操作是定义在SQLAlchemy中，不是在数据库里的，当更新一条数据时设置，大部分用于updateTime这类字段 |
| autoincrement    | 设置为整型自动增长，只有没有默认值，并且是Integer类型，默认是True |
| quote            | 如果列明是关键字，则强制转义，默认False                      |

### 全部类型

在上面Column类中type_是专门定义列的类型的，所有的类型都在sqlalchemy中的 `types.py` 文件里面，通过 `from sqlalchemy import types` 即可看到，所有类型内容如下：

```python
__all__ = ['TypeEngine', 'TypeDecorator', 'UserDefinedType', 'ExternalType', 'INT', 'CHAR', 'VARCHAR', 'NCHAR', 'NVARCHAR', 'TEXT', 'Text', 'FLOAT', 'NUMERIC', 'REAL', 'DECIMAL', 'TIMESTAMP', 'DATETIME', 'CLOB', 'BLOB', 'BINARY', 'VARBINARY', 'BOOLEAN', 'BIGINT', 'SMALLINT', 'INTEGER', 'DATE', 'TIME', 'TupleType', 'String', 'Integer', 'SmallInteger', 'BigInteger', 'Numeric', 'Float', 'DateTime', 'Date', 'Time', 'LargeBinary', 'Boolean', 'Unicode', 'Concatenable', 'UnicodeText', 'PickleType', 'Interval', 'Enum', 'Indexable', 'ARRAY', 'JSON']
```

**如果需要使用上述类型，则需从 `sqlalchemy` 进行导入。**例如：

```python
from sqlalchemy import Integer, String
```

## 建表映射

### 新建数据库表

首先我们使用上面的方法通过sql语句来创建数据库表：

```python
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
with engine.connect() as conn:
    sql = '''create table `user_account`(
             id int primary key,
             name varchar(30),
             fullname varchar(100));'''
    conn.execute(sql)
```

**前面提到首先需要创建基本结构 Table 表对象来对应操作的数据库表。因此需要一个集合放置表的信息，称为 MetaData 对象，本质上是一个 facade 在Python字典中存储了一系列 Table 对象键控到其字符串名称。有了 MetaData 对象，我们可以声明一些 Table 表对象，即表的模型。** 例如，它将表示网站的用户和表 address，表示与中的行关联的电子邮件地址列表 user 表等：

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
# 生成MetaData对象
metadata_obj = MetaData(engine)
# 建立数据表
user_table = Table(
    "user",  # 数据库表名称
    metadata_obj,  # MetaData对象
    Column('id', Integer, primary_key=True),
    Column('name', String(30)),
    Column('fullname', String(100))
)
# 安全创建表（操作会先判断表是否存在，存在不创建，否则创建）
metadata_obj.create_all(engine)
```

!> 注意：`"user"` 数据库表名称和 `metadata_obj` MetaData对象位置是固定的，不可交换。

可以看到 Table 很像SQL CREATE TABLE语句；从表名开始，然后列出每列，其中每列都有一个名称和一个数据类型。上面使用的对象是：

- `Table` -表示数据库表。

- `Column` -表示数据库表中的列，通常包括一个字符串名和一个类型对象。

  ```python
  # 存储Column对象的父对象Table通常位于Table.c
  user_table.c.name  # Column('name', String(length=30), table=<user_account>)
  user_table.c.keys()  # ['id', 'name', 'fullname']
  ```

- `Integer`、`String` -表示SQL数据类型， 例如给“name”列指定长度为30就写为了 `String(30)`

### 声明简单约束

在上面的建表中，`primary_key=True` 表示该 `Column` 是此表的主键的一部分。除了主键约束声明外，还有外键约束声明、非空约束等。例如创建下面的表：

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, ForeignKey

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
# 生成MetaData对象
metadata_obj = MetaData(engine)
# 建立数据表
address_table = Table(
    "address",
    metadata_obj,
    Column('id', Integer, primary_key=True),
    Column('user_id', ForeignKey('user.id'), nullable=False),  # 关联外键user表id列，非空
    Column('email_address', String(100), nullable=False)  # 非空
)
# 安全创建表
metadata_obj.create_all(engine)
```

**create 进程还负责按照正确的顺序发出create语句，上面外键约束依赖于 `user` 表已存在，因此是先创建 `user` 表后创建 `address` 表。**在更复杂的依赖场景中，外键约束也可以在使用ALTER之后应用于表。

?> 当使用 `ForeignKey` 对象可以省略该 Column 数据类型，它可以从相关列的值推断出来的，在上面的示例中 `Integer` 的数据类型 `user.id` 列。

?> MetaData 对象除了有 `MetaData.create_all()` 安全创建表方法，还有 `MetaData.drop_all()` 删除表方法，该方法将以删除架构元素而发出CREATE相反的顺序发出DROP语句。

### 声明映射类

上面我们建立好了 `user`、`address` 表，**现在我们可以依据这两张表来映射类了，映射类可以在ORM持久性和查询操作中使用。**映射内容如下：

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")

# declarative_base()是一个工厂函数，它为声明性类定义构造基类。
Base = declarative_base()

# Base作为我们声明的ORM映射类的基类。
class User(Base):
    # user表
    __tablename__ = 'user'
    # id字段，主键
    id = Column(Integer, primary_key=True)
    # name姓名字段
    name = Column(String(30))
    # fullname全名字段
    fullname = Column(String(100))

class Address(Base):
    # address表
    __tablename__ = 'address'
    # id字段，主键
    id = Column(Integer, primary_key=True)
    # user_id字段
    user_id = Column(Integer, ForeignKey('user.id'), nullable=False)
    # email_address字段
    email_address = Column(String(100), nullable=False)
```

**这里强调一下，并不是必须先有 `user`、`address` 表才能映射类，它们只是一个依据。假如没有 `user`、`address` 表，我们可以先定义好映射类，根据映射类来创建表。**使用下面方法：

```python
# 使用映射类来创建表
Base.metadata.create_all(engine)
```

## CRUD

### 创建会话

在最开始我们和数据库成功建立了连接，还进行了查询操作、建立了数据表，但这只是常规操作的一小部分，还有更新、插入等操作，但这些操作就比较特殊了，**因为查询、建表的操作要么成功要么失败，而更新、插入操作有可能成功一半后失败，因此我们需要一个原子性的操作对象，而这个对象就是session。**

- Atomicity（原子性）：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

ORM通过session与数据库建立连接进行通信，如下所示：**通过sessionmake方法创建一个Session工厂，然后在调用工厂的方法来实例化一个Session对象。**

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
# 实例化session对象
DBSession = sessionmaker(bind=engine)
session = DBSession()
```

### 插入操作

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
# 实例化session对象
DBSession = sessionmaker(bind=engine)
session = DBSession()

# declarative_base()是一个工厂函数，它为声明性类定义构造基类。
Base = declarative_base()

# Base作为我们声明的ORM映射类的基类
class User(Base):
    # user表
    __tablename__ = 'user'
    # id字段，主键
    id = Column(Integer, primary_key=True)
    # name姓名字段
    name = Column(String(30))
    # fullname全名字段
    fullname = Column(String(100))

# 实例化对象
User1 = User(id=1001, name='ling', fullname="ling jing")
User2 = User(id=1002, name='molin', fullname="molin xi")
User3 = User(id=1003, name='karl', fullname="karl zhou")

# 添加对象
session.add_all([User1, User2, User3])
# 提交session操作
session.commit()
# 关闭session
session.close()
```

### 查询操作

查询操作是CRUD里面最为复杂，最为繁琐的一个步骤。**在Sqlalchemy中，数据库的查询操作是通过Query对象来实现的。Session提供了创建Query对象的接口。Query对象返回的结果是一组同一映射（Identity Map）对象组成的集合。所谓同一映射，是指每个对象有一个唯一的ID，如果两个对象（的引用）ID相同，则认为它们对应的是相同的对象。事实上，集合中的一个对象，对应于数据库表中的一行（即一条记录）。**

查询操作格式：

```
session.query(映射类名).过滤条件.返回形式
```

查询操作例子：

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data")
# 实例化session对象
DBSession = sessionmaker(bind=engine)
session = DBSession()

# declarative_base()是一个工厂函数，它为声明性类定义构造基类。
Base = declarative_base()

# Base作为我们声明的ORM映射类的基类。
class User(Base):
    # user表
    __tablename__ = 'user'
    # id字段，主键
    id = Column(Integer, primary_key=True)
    # name姓名字段
    name = Column(String(30))
    # fullname全名字段
    fullname = Column(String(100))

# filter方法可以像sql那样写where条件使用>、<等符号，但引用列名时，需要通过`类名.属性名`的方式，等号用==
user_1 = session.query(User).filter(User.id==1001).first()
print(user_1)           # <__main__.User object at 0x000001F71E244508>
print(user_1.name)      # ling
print(user_1.fullname)  # ling jing
# filter_by方法的where条件不能使用>、<等符号，指定列名时，参数名即对应名类中的属性名，等号用=
user_2 = session.query(User).filter_by(id=1002).first()
print(user_2)           # <__main__.User object at 0x000001F71E244D48>
print(user_2.name)      # molin
print(user_2.fullname)  # molin xi
'''
注释：可以看到查询出来的数据sqlalchemy直接给映射成一个对象了，这个对象和我们创建表时候的class是一致的，我们就也可以直接通过对象的属性就可以直接调用就可以了。
'''
```

过滤条件如下表格：

| 过滤条件                                      | 含义                                           |
| --------------------------------------------- | ---------------------------------------------- |
| filter(类.id==1001)                           | 筛选id为1001的数据                             |
| filter(类.id==None)                           | 筛选id为空的数据                               |
| filter(类.id!=1001)                           | 筛选id不为1001的数据                           |
| filter(类.name.like("%m%"))                   | 模糊查询name中有m字符的数据（区分大小写）      |
| filter(类.name.ilike("%m%"))                  | 模糊查询name中有m或M字符的数据（不区分大小写） |
| filter(类.name.contains("m"))                 | 模糊查询name中有m字符的数据                    |
| filter(and_(类.id>=1001, 类.name.like("%m%")) | 筛选id大于等于1001且name中有m字符的数据        |
| filter(or_(类.id>=1001, 类.name.like("%m%"))  | 筛选id大于等于1001或name中有m字符的数据        |
| filter(类.id.in_([1001, 1002]))               | 筛选id等于1001或1002的数据                     |
| filter(类.id.notin_([1001, 1002]))            | 筛选id不等于1001且1002的数据                   |
| filter(类.id!=1001).limit(3)                  | 筛选id不为1001最前面3条数据                    |



