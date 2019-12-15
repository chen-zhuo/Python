# `Python` 内存、数据类型

### `Python` 内存

**一个数据存储在计算机中必定有 `内存地址(id)`、`类型(type)`、`值(value)` 这三个属性。**

##### 内存地址

`Python` 会为每个对象分配内存，每个内存都有其对应的地址，因此每个对象都会有一个内存地址。

**内存地址**：`id(对象)`，查看对象的内存地址。

```python
print(id(4))	# 1389128816
print(id(4.0))  # 204967854920
print(id('4'))  # 204702041344
print(id(4))	# 1389128816

# 变量作为存储数据对象的容器，其内存地址必然和数据对象的内存地址一样。
a = 1
print(id(1), id(a))    # 1965911056 1965911056

b = [1]
print(id([1]), id(b))  # 39344487560 39344487944
```

##### 关键字 `is`

**`is` 判断两个变量的 `id` 是否相同，即是否指向同一个对象；`==` 判断两个变量的值是否相等。**

```python
a = 2.0
b = 2
print(a is b) 	# False
print(a == b)   # True
```

##### 可变对象

内存地址(`id`)不变的情况下，可以被改变对象称之为**可变对象**，通俗点说就是**原地改变**。

Python中列表(`list`)、字典(`dict`)、集合`set`)都属于可变对象。

```python
list_1 = [4]
print(list_1, id(list_1))	# [4] 	  927911656968
list_1.append(5)
print(list_1, id(list_1))	# [4, 5]  927911656968

dict_1 = {'name': 'chen'}
print(dict_1, id(dict_1))	# {'name': 'chen'}            927909112784
dict_1['age'] = 18
print(dict_1, id(dict_1))	# {'name': 'chen', 'age': 18} 927909112784
```

除了切片、拷贝之类的其他操作，都不会改变可变对象的数量，可变对象有且只有一个。

```python
list_1 = [4]
print(list_1, id(list_1))	# [4] 	    1095727791624

a = list_1
a.append(5)
print(a, id(a))	            # [4, 5]    1095727791624，内存地址一样，对象还是list_1

b = list_1
b.append(6)
print(b, id(b))	            # [4, 5, 6] 1095727791624，内存地址一样，对象还是list_1
print(a, id(a))	            # [4, 5, 6] 1095727791624，内存地址一样，对象还是list_1

list_2 = list_1[:]	        # 这里出现了切片，产生新对象
print(list_2, id(list_2))	# [4] 	    871593557128，内存地址不一样，list_2是新对象
```

##### 不可变对象

内存地址(`id`)不变的情况下，不能被改变对象称之为**不可变对象**，通俗点说就是**原地不动**。

`Python` 中整型(`int`)、浮点型(`float`)、字符型(`str`)、元组(`tuple`)都属于不可变对象。

```python
a = 10
print(a, id(a))		# 10 1965911344
a += 1
print(a, id(a))		# 11 1965911376
# 注释：修改变量时，由于值的类型属于不可变对象，所以会开辟新的内存地址，把值复制一份放到新的内存地址后再改变，再让变量重新指向这个新的地址。
```

除拷贝之类的其他操作，都会改变不可变对象的数量。

```python
a = 10
print(a, id(a))  # 10 1965911344

b = a
b += 1
print(b, id(b))  # 11 1965911376

c = a
c += 2
print(c, id(c))  # 12 1965911408

d = a
print(d, id(d))  # 10 1965911344
```

##### 赋值

赋值： `=`，**赋值的过程就是拷贝对象的地址，增加指向同一地址的变量数量**（当对象改变时，所有指向同一对象的变量都发生改变）。

```python
# alist、b为变量，[1,2,3,["a","b"]]为对象
alist=[1,2,3,["a","b"]]
b=alist
print(b)		# [1, 2, 3, ['a', 'b']]

alist.append(5)
print(alist)	# [1, 2, 3, ['a', 'b'], 5]
print(b)		# [1, 2, 3, ['a', 'b'], 5]
```

![img](https://img2018.cnblogs.com/blog/546188/201810/546188-20181031094701785-1018065288.png)

##### 浅拷贝

**浅拷贝**：`copy.copy`，在嵌套的情况下，**最外层对象拷贝形成新的对象，内层对象就拷贝对象的地址**。

通俗点讲，浅拷贝生成的新对象和原对象的关系就像是连体婴儿，有相互独立的部分，又有相互影响的部分。

```python
import copy

alist=[1,2,3,["a","b"]]

c=copy.copy(alist)      
print(alist)		    # [1, 2, 3, ['a', 'b']]
print(c)	            # [1, 2, 3, ['a', 'b']]（浅拷贝，形成新对象）

alist.append(5)			# 最外层对象添加'5'
print(alist)		    # [1, 2, 3, ['a', 'b'], 5]
print(c)		        # [1, 2, 3, ['a', 'b']]

alist[3].append('cccc') # 内层对象添加'cccc'
print(alist)		    # [1, 2, 3, ['a', 'b', 'cccc'], 5]
print(c)		        # [1, 2, 3, ['a', 'b', 'cccc']]
```

![img](https://img2018.cnblogs.com/blog/546188/201810/546188-20181031095328930-1590606034.png)

##### 深拷贝

深拷贝：`copy.deepcopy`，就是在新的内存中拷贝出一个与原对象完全一样的对象，两个对象相互对立。

通俗点讲，深拷贝就是克隆出一个新个体，新个体和原个体之间互不冲突，互不影响。

```python
import copy

alist=[1,2,3,["a","b"]]
d=copy.deepcopy(alist)
print(alist)		     # [1, 2, 3, ['a', 'b']]
print(d)		         # [1, 2, 3, ['a', 'b']]

alist.append(5)
print(alist)		     # [1, 2, 3, ['a', 'b'], 5]
print(d)		         # [1, 2, 3, ['a', 'b']]

alist[3].append("ccccc")
print(alist)		     # [1, 2, 3, ['a', 'b', 'ccccc'], 5]
print(d)		         # [1, 2, 3, ['a', 'b']]
```

![img](https://img2018.cnblogs.com/blog/546188/201810/546188-20181031095505004-75839266.png)

**注意：赋值、浅拷贝、深拷贝对于可变对象来说，只有赋值不会产生新的对象，其他操作会有新对象产生。**

**注意：赋值、浅拷贝、深拷贝对于不可变对象来说，所产生的新变量都是指向原对象的，没有新对象产生。**

### 数据类型

##### 八大类型

**容器类型**：就是能通过循环遍历的数据类型就是序列，不能通过循环遍历的数据类型就是非序列。

**下标索引**：就是能使用下标的数据类型就是有序的，不能使用下标的数据类型就是无序的。

**注意：Python 没有 `char` 类型 和 `byte` 类型。**

|  类型  | 关键字  |  对象类型  | 容器类型 | 下标索引 |      特点       |
| :----: | :-----: | :--------: | :------: | :------: | :-------------: |
|  整型  |  `int`  | 不可变对象 |  非序列  |   无序   |    只有整数     |
| 浮点型 | `float` | 不可变对象 |  非序列  |   无序   |    存在小数     |
| 字符型 |  `str`  | 不可变对象 |   序列   |   有序   |    所有字符     |
| 布尔型 | `bool`  | 不可变对象 |  非序列  |   无序   | `True`、`Flase` |
|  列表  | `list`  |  可变对象  |   序列   |   有序   |  操作自由度高   |
|  字典  | `dict`  |  可变对象  |   序列   |   无序   |   键值对存储    |
|  集合  |  `set`  |  可变对象  |   序列   |   有序   |    自动去重     |
|  元祖  | `tuple` | 不可变对象 |   序列   |   有序   |  内容不可改变   |

##### 类型转换函数

`Python` 提供了一系列将变量或值进行类型转换的**内置函数**，将要转换的数据添加到函数中进行转换。

**`int` 函数**：将对象类型转换成整型。

1. 浮点型转整型，只保留整数部分。
2. 布尔型转整型，`True = 1`、`False = 0`。
3. 字符型转整型，字符串**只能包含数字**才能转换成整型。

```python
print(int(12.9))   # 12，
print(int(True))   # 1
print(int(False))  # 0
print(int('12'))   # 12 
print(int('+12'))  # 12
print(int('-12'))  # -12
print(int('a'))    # 错误，a不是数字
```

**`float` 函数**：将对象类型转换成浮点型。

1. 整型转浮点型，在整数后面加一个 `.0`。
2. 布尔型转浮点型，`True = 1.0`、`False = 0.0`。
3. 字符型转浮点型，字符串**只能包含数字**才能转换成浮点型。

```python
print(float(123))        # 123.0
print(float(False))      # 0.0
print(float(True))       # 1.0
print(float('100.23'))   # 100.23
print(float('-100'))     # -100
print(float('+100'))     # 100
print(float('a'))        # 错误，a不是数字
```

**`str` 函数**：将对象类型转换成字符型。

1. **任何类型的数据都可以转换成字符串**。
2. **其他数据转换成字符串的时候，就是直接在数据的外层加引号**。

```python
print(str(2))     # 2，结果不显示引号
print(str(2.0))   # 2.0
print(str(True))  # True
print(str(['a'])) # ['a']
```

**`bool` 函数**：将对象类型转换成布尔型。

1. **任何类型的数据都可以转换成布尔值。**
2. **所有为0、为空的值在布尔类型中都为 `False`，其他的值都为 `True`。**

```python
print(bool(0))       # False
print(bool(0.0))     # False
print(bool(1.0))     # True
print(bool(''))      # False
print(bool('cz'))    # True
```

**`list` 函数**：将对象类型转换成列表。

1. 字符型可以转成列表。
2. 字典可以转成列表。
3. 集合可以转成列表。

```python
lis = list('abc123')
print(lis, type(lis))    # ['a', 'b', 'c', '1', '2', '3'] <class 'list'>

set3 = {12, 'abc', 'hh', 32}
print(list(set3))  # [32, 'abc', 'hh', 12] 列表
```

**`dict` 函数**：将对象类型转换成字典。

1. 元祖可以转成字典。

```python
print(dict([(1, 2), (3, 4)]))			# {1: 2, 3: 4}
print(dict(([1, 2], [3, 4])))			# {1: 2, 3: 4} 
print(dict(('a','b'),(1,2)))			# 报错，参数错误
print(dict(zip(('a','b'),(1,2))))		#{'a': 1, 'b': 2}
```

**`set` 函数**：将对象类型转换成集合。

1. 字符型可以转成集合。
2. 列表可以转成集合。

```python
set2 = set('abc123')  
print(set2)       # {'a', 'b', '2', '1', '3', 'c'}
set3 = [12, 'abc', 'hh', 32]
print(set(set3))   # {32, 'abc', 'hh', 12} 集合
```

**`tuple` 函数**：将对象类型转换成元祖。

1. 列表可以转成元祖。

```python
print(tuple([1,2,3])) # (1, 2, 3)
```

**`eval` 函数**：用来执行一个字符串表达式，并返回表达式的值。

```python
a = '[1,2,3]'
a1 = eval(a)
print(a1,type(a1))   # [1, 2, 3] <class 'list'>

b ='{"a":1, "b":2}'
b1 = eval(b)
print(b1,type(b1))   # {'a': 1, 'b': 2} <class 'dict'>

c = '(1,2,3)'
c1 = eval(c)
print(c1,type(c1))   # (1, 2, 3) <class 'tuple'>

d = '2+2'
d1 = eval(d)
print(d1,type(d1))   # 4 <class 'int'>
```

##### 类型查看判断

`type(data)`：查看 `data` 数据类型

```python
print(type(10))     # int
print(type('abc'))  # str
```

`isinstance(值，类型)` ：判断值是否是指定类型，是结果为 `True`，否则结果为 `False`。

```python
print(isinstance(10, int))    # True
print(isinstance(12.0, int))  # False
```

##### 类型区别

对于不可变对象来说赋值、浅拷贝、深拷贝所产生的新变量都是指向原对象的，并没有新对象产生。

```python
from copy import copy, deepcopy
a = 1
b = a
c = copy(a)
d = deepcopy(a)
print(id(a))	# 1978231824
print(id(b))	# 1978231824
print(id(c))	# 1978231824
print(id(d))	# 1978231824
```

