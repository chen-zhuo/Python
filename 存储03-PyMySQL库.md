# PyMySQL库

这里我们开始学习如何使用Python来连接数据库、将数据存储在数据库中以及对数据库中的数据进行CRUD（增删改查）操作。

**首先介绍一下，PyMySQL库是一个专门用于连接并操作MySQL数据库的第三方库。**

## 准备工作

### 服务启动

在执行下面操作时，确保电脑中安装了MySQL，且将MySQL的路径添加到了环境变量中，然后再在命令行窗口输入下面命令，启动MySQL服务：

```
net start mysql
```

### 安装PyMySQL

安装要求：

```
Python以下之一:
CPython >= 3.6
Latest PyPy 3
MySQLServer以下之一:
MySQL >= 5.6
MariaDB >= 10.0
```

安装PyMySQL：

```
pip install pymysql
```

## 操作MySQL

### 连接MySQL

安装好了PyMySQL库，首先第一步就是要连接到MySQL数据库上：

```python
# 导入pymysql
import pymysql

# 主机地址
host = '127.0.0.1'
# 用户名
user = 'root'
# 密码
password = '123456'
# 数据库名称
database = 'lb'
# MySQL数据库的默认端口
port = 3306
# 生成连接db对象
db = pymysql.connect(host, user, password, database, charset='utf8', port=port)
# 生游标cursor操作对象
cursor = db.cursor()
# 操作MySQL
...
# 提交操作
db.commit()
# 关闭MySQL连接
db.close()

'''
注释：
1.db对象负责开启\关闭连接MySQL数据库、提交执行结果，游标cursor对象负责操作数据库。
2.使用时将MySQL的连接对象、操作游标对象定义成全局变量，方便函数调用。
3.cursor和db.cursor()是有区别的，使用cursor去执行sql语句，自始至终只有一个操作游标，如果使用db.cursor()去执行sql语句，则会生成多个操作游标，而且还可能会报错。
4.除了查询操作外的其他操作都需要进行db.commit()提交，否则操作MySQL无效。
5.db.close()关闭MySQL连接不要写在循环中，若频繁的关闭数据库，可能会导致数据库崩溃。
6.添加cursorclass=pymysql.cursors.DictCursor参数，可以以字典形式返回结果的游标。

常见错误：
(1049, Unknown database 'lb') # 没有名称为lb的数据库
(1045, Access denied for user 'rooot'@'localhost'(using password: YES)) # 密码或用户名错误
'''
```

我们更加建议使用如下写法：

```python
# 导入pymysql
import pymysql

# 连接数据库
connection = pymysql.connect(host='127.0.0.1',
                             user='root',
                             password='123456',
                             database='lb',
                             port = 3306)

with connection.cursor() as cursor:
    # 操作MySQL
    ...
'''
注释：这样写的好处就是可以不用写db.close()，因为执行结束就自动关闭了，同时也排除了生成多个cursor的可能性。
'''
```

### 执行SQL语句

```python
# SQL建表语句：创建一张名为USER的数据表，表中三个字段，id字段为主键、自增型，NAME字段为字符串类型最大长度为10，HOBBY字段为字符串类型最大长度为100。
sql1 = "CREATE TABLE USER( Id INT PRIMARY KEY AUTO_INCREMENT, \
						  NAME VARCHAR(10), \
						  HOBBY VARCHAR(100)\
						 )"
# 操作游标cursor使用execute方法执行sql1中的SQL语句
cursor.execute(sql1)

# SQL插入语句：向USER表中的NAME、HOBBY字段插入变量name、hobby的值(%s代表插入的变量是字符型)
sql2 = 'INSERT INTO USER(NAME, HOBBY) VALUES("%s", "%s")' % ('chen', 'sport')
# 执行sql2中的SQL语句
cursor.execute(sql2)

# SQL插入语句：向USER表中的NAME、HOBBY字段中插入值为"zhuo"、"computer"的字符串
sql3 = "INSERT INTO USER(NAME, HOBBY) VALUES('zhuo', 'computer')"
# 执行sql3中的SQL语句
cursor.execute(sql3)

'''
executemany(query, args)
参数：	
    query(str) -- 要执行的sql。
    args(tuple或list) -- 序列或映射的序列。它用作参数。
注释：此方法可以提高多行INSERT和REPLACE的性能。
'''

# 使用连接对象db提交要执行的操作数据，若不提交，以上操作全部无效
db.commit()
```

### 动态数据插入

```python
# 数据的插入是通过SQL语句实现的
sql = 'INSERT INTO students(id, name, age) values(%s, %s, %s)

# 有一个极其不方便的地方，比如突然增加了某一个字段sex，此时SQL语句就要改成：
sql = 'INSERT INTO students(id, name, age, sex) values(%s, %s, %s, %s)

# 这显然不是我们想要，这时我们就需要一个通用的插入动态变化的字典的方法：
table = 'students'
data = {
        'id': 1,
        'name': 'chen',
        'age': 20
    	}
keys = ','.join(data.keys())
values = ','.join(['%s'] * len(data))

sql = f'INSERT INTO {table}({keys}) VALUES({values})'
    
try:
   if cursor.execute(sql, tuple(data.values())):
        # 尝试执行SQL语句，如果成功提交操作数据
     	print('Successful')
        db.commit()
except:
   # 操作失败就回滚再次执行SQL语句
   print('Failed')
   db.rollback()
```

### 返回执行结果

```python
# sql查询语句
sql = f'SELECT cookie FROM 表名 WHERE 条件语句'
# 执行sql
cursor.execute(sql)
# fetchall()以元组的形式返回所有的结果
res = cursor.fetchall()
print(res)     # ((数据1), (数据2), (数据3)...)
print(res[0])  # (数据1)
# fetchmany(size=None)以元组的形式返回所有指定长度的结果
res = cursor.fetchmany(size=3)
print(res)     # ((数据1), (数据2), (数据3))
print(res[0])  # (数据1)
# fetchone()以元组的形式返回一条结果，如果没有结果返回None
res = cursor.fetchone()
print(res)     # (数据1)
print(res[0])  # 数据1
```
