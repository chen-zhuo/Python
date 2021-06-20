# Pandas

pandas是基于numpy的一种工具，该工具是为了解决数据分析任务而创建。

pandas纳入了大量库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具
   pandas提供了大量能使我们快捷便捷处理数据的函数和方法，数据结构是series:一维数组，与numpy的一维数组array类似。二者与python基本的数据结构list也很相近
   其区别是list中的元素可以是不同的数据类型，而array和series中则只允许存储相同的数据类型，这样可以更加有效的使用内存，提高运算效率

### Numpy 和Pandas

**Numpy更适合处理统一的数值数组数据**

**Pandas则专门处理表格和混杂数据**

### 数据结构

Pandas的两个主要数据结构：Series和DataFrame

### Series

Series是⼀种类似于**⼀维数组的对象**，它由⼀组数据（各种NumPy数据类型）以及⼀组与之相
关的数据标签（即索引）组成

```
# 把pandas重命名为pd
import pandas as pd

obj = pd.Series([4, 7, -5, 3])
print(obj)
print('+++++++++++++++++')
# 打印obj的值
print(obj.values)
print('+++++++++++++++++')
# 打印obj的索引范围
print(obj.index)
print('+++++++++++++++++')
# 每个Series对象都有一个name属性，初始为‘None’空
print(obj.name)
# 给obj这个Series对象name属性赋值为'attrs'
obj.name = 'attrs'
print(obj.name)
print('+++++++++++++++++')
print(obj)

结果
0    4
1    7
2   -5
3    3
# 自动添加打印类型
dtype: int64
+++++++++++++++++
[ 4  7 -5  3]
+++++++++++++++++
RangeIndex(start=0, stop=4, step=1)
+++++++++++++++++
None
attrs
+++++++++++++++++
0    4
1    7
2   -5
3    3
# 再次打印obj，就自动添加了name属性
Name: attrs, dtype: int64
```

##### 自建索引

```
import pandas as pd

obj = pd.Series([4, 7, -5, 3])
# 自建索引序列
obj.index = ['d', 'b', 'a', 'c']
print(obj)
print('+++++++++++++++++')
# 打印对应‘a’序列的值
print(obj['a'])
print('+++++++++++++++++')
print(obj.index)

结果：
d    4
b    7
a   -5
c    3
dtype: int64
+++++++++++++++++
-5
+++++++++++++++++
Index(['d', 'b', 'a', 'c'], dtype='object')
```

##### 索引使用

```
import pandas as pd

obj = pd.Series([4, 7, -5, 3])
obj.index = ['d', 'b', 'a', 'c']

# 将索引‘d’对应的值修改为6
obj['d'] = 6
# 添加索引‘e’值为9
obj['e'] = 9

# 只打印索引'a', 'd', 'e'的值，注意这里面是两个[]
print(obj[['a', 'd', 'e']])
print('+++++++++++++++++')
# 打印值是否大于0，True和False
print(obj > 0)
print('+++++++++++++++++')
# 只打印大于0的索引和值
print(obj[obj > 0])
print('+++++++++++++++++')
# 打印索引和乘以2的值
print(obj*2)
print('+++++++++++++++++')
# 判断‘e’是否在obj
print('e' in obj)
# 判断‘f’是否在obj
print('f' in obj)

结果：
a   -5
d    6
e    9
dtype: int64
+++++++++++++++++
d     True
b     True
a    False
c     True
e     True
dtype: bool
+++++++++++++++++
d    6
b    7
c    3
e    9
dtype: int64
+++++++++++++++++
d    12
b    14
a   -10
c     6
e    18
dtype: int64
+++++++++++++++++
True
False
```

##### 字典创建Series

```
import pandas as pd

sdata = {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000}
# 用sdata字典创建Series
obj3 = pd.Series(sdata)
print(obj3)
print('+++++++++++++++++')

# obj4中重新赋予序列
# 'California'为新加入的序列，值为‘NaN’空
states = ['California', 'Ohio', 'Oregon', 'Texas']
obj4 = pd.Series(sdata, index=states)
print(obj4)
print('+++++++++++++++++')

# 将共有序列的值相加，其他序列的值为‘NaN’
print(obj3+obj4)
print('+++++++++++++++++')

# isnull函数可⽤于检测缺失数据，notnull值结果相反
obj5 = pd.isnull(obj4)
print(obj5)

结果：
Ohio      35000
Texas     71000
Oregon    16000
Utah       5000
dtype: int64
+++++++++++++++++
# NaN（即“⾮数字”（not a number），在pandas中，它⽤于表示缺失或NA值）
California        NaN
Ohio          35000.0
Oregon        16000.0
Texas         71000.0
dtype: float64
+++++++++++++++++
California         NaN
Ohio           70000.0
Oregon         32000.0
Texas         142000.0
Utah               NaN
dtype: float64
+++++++++++++++++
California     True
Ohio          False
Oregon        False
Texas         False
dtype: bool
```

### DataFrame

DataFrame是⼀个**表格型的数据结构**，它含有⼀组有序的列，每列可以是不同的值类型（数
值、字符串、布尔值等），DataFrame既有行索引也有列索引。

建立DataFrame, 传⼊⼀个由**等长**列表或NumPy数组组成的字典

```
import pandas as pd

data = {'state': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada', 'Nevada'],
        'year': [2000, 2001, 2002, 2001, 2002, 2003],
        'pop': [1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}
frame = pd.DataFrame(data)
print(frame)
print('++++++++++++++++++++')

# 指定打印列的序列
frame1 = pd.DataFrame(data, columns=['year', 'state', 'pop'])
print(frame1)
print('++++++++++++++++++++')

# 新加入debt列（初始值为NaN），自建索引
frame2 = pd.DataFrame(data, columns=['year', 'state', 'pop', 'debt'],
                            index=['one', 'two', 'three', 'four', 'five', 'six'])
print(frame2)
print('++++++++++++++++++++')

# 打印指定year列
print(frame2.year)
print('++++++++++++++++++++')

# 打印指定索引‘one’行，注意前面是.loc
print(frame2.loc['one'])

结果：
    state  year  pop
0    Ohio  2000  1.5
1    Ohio  2001  1.7
2    Ohio  2002  3.6
3  Nevada  2001  2.4
4  Nevada  2002  2.9
5  Nevada  2003  3.2
++++++++++++++++++++
   year   state  pop
0  2000    Ohio  1.5
1  2001    Ohio  1.7
2  2002    Ohio  3.6
3  2001  Nevada  2.4
4  2002  Nevada  2.9
5  2003  Nevada  3.2
++++++++++++++++++++
       year   state  pop debt
one    2000    Ohio  1.5  NaN
two    2001    Ohio  1.7  NaN
three  2002    Ohio  3.6  NaN
four   2001  Nevada  2.4  NaN
five   2002  Nevada  2.9  NaN
six    2003  Nevada  3.2  NaN
++++++++++++++++++++
one      2000
two      2001
three    2002
four     2001
five     2002
six      2003
Name: year, dtype: int64
++++++++++++++++++++
year     2000
state    Ohio
pop       1.5
debt      NaN
Name: one, dtype: object
```

