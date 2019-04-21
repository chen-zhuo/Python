# 元祖tuple

### 定义

**元祖(tuple)是Python中用来存储数据的一个数据类型。**

元祖里面能存储多个数据，里面存储的**单个数据**叫**元素**

### 特点

1. **使用()将元素包含起来，多个元素之间用逗号隔开**
2. **当只有一个元素的时候，后面要加逗号**
3. 元素的类型可以是**任何类型**
4. 元祖里面的元素**不能被修改的**

```
(a,)		(1, 2, 'abc')
```

**注意：元祖(tuple)就是不可变的列表，只有查操作可以使用，其他的改、增、删相关操作都不能作用于元祖**

### 元祖方法

##### 常规方法

下标、切片、遍历、长度、in/not in、拼接、重复操作都能作用于元祖

```
colors = ('red', 'green', 'yellow', 'purple')
print(type(colors))     # <class 'tuple'>

print(colors[1])        # green
print(colors[0:3])      # ('red', 'green', 'yellow')
print(colors[0::2])     # ('red', 'yellow')

for item in colors:
    print(item)
                        # red
                        # green
                        # yellow
                        # purple（紫色）
    
print(len(colors))      # 4

print('red' in colors)  # True

print((1, 2)+(3, 4))    # (1, 2, 3, 4)

print((1, 2) * 2)       # (1, 2, 1, 2)
```

##### 多变量获取

方法：变量1，变量2... = 元祖

作用：通过多个变量分别获取元祖元素（**变量个数和元祖元素个数一样,不一样会报错**）

方法：变量1，*变量2 = 元祖

作用：通过变量名前加*可以把变量变成列表，获取多个元素

```
names = ('name1', 'name2', 'name3',)
x, y, z = names  	            # 通过多个变量分别获取元祖元素(变量个数和元祖元素的个数必须一样，不一样会报错)
print(x, y, z)  	            # name1 name2 name3

names = ('name1', 'name2', 'name2_2', 'name2_3', 'name3')
first, *midel, last = names  	    # 通过变量名前加*可以把变量变成列表，获取多个元素
print(first, midel, last)  	    # name1 ['name2', 'name2_2', 'name2_3'] name3

*name1, name = names
print(name1, name)  	            # ['name1', 'name2', 'name2_2', 'name2_3'] name3

name, *name1 = names  	            # 让name获取第一个元素，剩下的部分作为列表name1的元素
print(name, name1)  	            # name1 ['name2', 'name2_2', 'name2_3', 'name3']
```

##### 元祖遍历

方法：*元祖
  
作用：将元祖里面的元素遍历

```
a = (10, 2)
print(*a, type(a))  # 10 2 <class 'tuple'>
```

### 计数器

```
from collections import Counter

def tuple_():
    tuple1 = (1, 1, 1, 7, 8, 4, 4, 7, 7, 7)
    tuple_count = Counter(tuple1)
    print(tuple_count)

if __name__ == '__main__':
    tuple_()
 
输出：
Counter({7: 4, 1: 3, 4: 2, 8: 1})
这里输出了列表和数组中某个数字出现的次数，并从多到少进行了排序
```
