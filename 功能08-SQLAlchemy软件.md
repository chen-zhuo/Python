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

本章节使用的SQLAlchemy版本为1.4，所有的例子也都适用于1.4以上的版本，有一点要注意的就是1.4版本与1.3版本有较大改动，以下案例或许不能通用。

### 安装检查

下载安装SQLAlchemy：

```
pip install SQLAlchemy
```

检查SQLAlchemy的安装版本：

```python
import sqlalchemy
print(sqlalchemy.__version__)  # 1.4.0
```

### 建立连接

**任何SQLAlchemy程序开始都是有一个只为特定数据库服务器创建一次的名为 `Engine` 的全局对象，此对象会通过一个URL字符串（该字符串将描述如何连接到数据库主机或后端）进行配置连接到特定数据库的中心源。**

```python
from sqlalchemy import create_engine

# 1.3.24版本
engine = create_engine("mysql+pymysql://用户名:密码@IP:端口/数据库名称", 
         max_overflow=10,  # 最大连接数
         pool_size=10,  # 连接池大小
         pool_timeout=60,  # 池中没有线程最多等待的时间，否则报错
         pool_recycle=60 * 60 * 2
        )

# 1.4.0版本
engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_data",
         echo=True,  # 将所有SQL记录到Python记录器中
         future=True  # 指定future参数为True以便我们充分利用2.0style
        )

# 从用户的角度来看Engine对象的唯一目的提供到数据库的连接单元，称为Connection。
conn = engine.connect()
# 如果我们希望将Engine对象的使用范围限制在特定上下文中，最好的方法是使用Python上下文管理器表单，也称为 the with statement。
with engine.connect() as conn:
    ...
```

### 执行文本SQL

`text()` 方法允许编写文本SQL语句。

`conn.execute()` 方法执行文本SQL语句。

```python
from sqlalchemy import text
from sqlalchemy import create_engine

engine = create_engine("mysql+pymysql://root:123456@127.0.0.1:3306/new_origin_database")

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

## 数据库元数据

