# 字典dict

### 定义

**字典（dict）是Python中用来存储数据的一个数据类型。**

字典里面能存储多个数据，所有数据**存储的形式**叫**键值对**。

键值对就是**一个键对应一个值**的模式，**左边为键，右边为值**，**即{key：value}，其中key对应键，value对应值（所存数据）**。

**注意：键值对中的键和值必须同时存在，任何一方消失，整个键值对就没有了。**

### 特点

1. **用 { } 表示一个字典，里面多个元素之间用逗号隔开。（没有元素表示 { } 空字典）**
2. **一个元素就是一个键值对，键值对中的键和值用冒号分开，左边为键，右边为值。**
3. **键，属于是不可变的数据类型，一般使用字符串作为key，而且键是唯一的。（若键不唯一，后面的值会覆盖前面的值）**
4. **值，value可以是任何类型的数据**
5. 字典可以增、删、改、查，但是是无序的(**不能使用下标**)

### 类型转换

元祖转字典

方法：dict（数据）

```
print(dict([(1, 2), (3, 4)]))			# {1: 2, 3: 4}
print(dict(([1, 2], [3, 4])))			# {1: 2, 3: 4} 
print(dict(('a','b'),(1,2)))			# 报错，参数错误
print(dict(zip(('a','b'),(1,2))))		#{'a': 1, 'b': 2}
```

字典转元祖

方法：字典.items()

作用：将字典中所有的键值对转换成一个一个的元祖，**key作为元祖的第一个元素，value作为元祖的第二个元素**

```
student_dict = {'name': '张三', 'score': {'english': 60, 'math': 100}}
print(student_dict.items())   

输出：
dict_items([('name', '张三'), ('score', {'english': 60, 'math': 100})])
```

**字典转布尔，只要有键值对的字典，其布尔型就为True；空字典布尔型为False。**

### 字典方法

##### 获取键

方法：字典.key**s**()

作用：获取字典**所有的key**，返回值的类型是dict_keys，**可以当成列表使用**。

```
student_dict = {'name': '张三', 'score': {'english': 60, 'math': 100}}

print(student_dict.keys())    # dict_keys(['name', 'score'])
```

##### 获取值

方法：字典.value**s**()

作用：获取字典**所有的value**，返回值的类型是dict_values，**可以当成列表使用**。

```
student_dict = {'name': '张三', 'score': {'english': 60, 'math': 100}}

print(student_dict.values())  # dict_values(['张三', {'english': 60, 'math': 100}])
```

##### 遍历字典

直接获取到所有的key，在获取对应的value

```
student_dict = {'name': '张三', 'study_id': 'py1805001', 'score': {'english': 60, 'math': 100}}
for key in student_dict:
    print(key, student_dict[key])

输出：
name 张三
study_id py1805001
score {'english': 60, 'math': 100}
```

##### 排序字典

方法：sorted(iterable, key=None, reverse=False)  

参数说明：

- iterable -- 可迭代对象。
- key -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
- reverse -- 排序规则，reverse = True 降序 ， reverse = False 升序（默认）。

```
a = dict(z=1, x=3, y=2)
print(a.items())		# dict_items([('z', 1), ('x', 3), ('y', 2)])
print(sorted(a.items()))# [('x', 3), ('y', 2), ('z', 1)]

# 键排序，用列表的形式返回
b1 = sorted(a.items(), key=lambda d: d[0], reverse=False) # 键升序
b2 = sorted(a.items(), key=lambda d: d[0], reverse=True) # 键降序
print(dict(b1))			# {'x': 3, 'y': 2, 'z': 1}
print(dict(b2))			# {'z': 1, 'y': 2, 'x': 3}

# 值排序，用列表的形式返回
c1 = sorted(a.items(), key=lambda d: d[1], reverse=False) # 值升序
c2 = sorted(a.items(), key=lambda d: d[1], reverse=True) # 值降序
print(dict(c1))			# {'z': 1, 'y': 2, 'x': 3}
print(dict(c2))			# {'x': 3, 'y': 2, 'z': 1}
```

##### 自建字典

方法：dict.fromkeys(序列，value)

作用：创建一个新的字典，序列中的元素作为key,value作为值

```
new_dict1 = dict.fromkeys('abc', '100')
print(new_dict1)  # {'a': '100', 'b': '100', 'c': '100'}

new_dict1 = dict.fromkeys(range(10), 100)
print(new_dict1)  # {0: 100, 1: 100, 2: 100, 3: 100, 4: 100, 5: 100, 6: 100, 7: 100, 8: 100, 9: 100}

new_dict1 = dict.fromkeys(['abc', 'dcc', '123'], '100')
print(new_dict1)  # {'abc': '100', 'dcc': '100', '123': '100'}
```

##### 更新字典

方法：字典1.update(字典2)

作用：使用字典2的键值对更新字典1的键值对。**若字典2中对应的键值对在字典1中不存在，就添加该键值对，存在就更新该键值对。**

```
dict1 = {'1': 'a', '2': 'b'}
dict1.update({'1': 'aaa', '3': 'bbb'})
print(dict1)  # {'1': 'aaa', '2': 'b', '3': 'bbb'}
```

### 增删改查

##### 查找

方法：字典[key]

作用：通过字典中key来获取value

```
person = {'name': '路飞', 'age': 17, 'face': 90}
print(person['name'])     # 路飞
print(person['face'])     # 90	 如果key不存在，会报错KeyError
```

方法：字典.get(key, default=None)

作用：通过字典中key来获取value，default如果指定键的值不存在时，返回该默认值值。

例子1：

```
person = {'name': '路飞', 'age': 17, 'face': 90}
print(person.get('name'))  		  # 路飞
print(person.get('aaa'))   		  # None  key不存在，返回默认None
print(person.get('aaa', 100))   # 100  key不存在，返回100
```

**注意：如果key值确定存在，使用[]语法去获取值。不确定key值是否存在才使用get方法去获取值**

方法：key  in/not in  字典

作用：判断字典中是否存在指定的key（只能判断key，不能判断value）

```
dog_dict = {'color': 'white', 'age': 3, 'type': '土狗'}
print('color' in dog_dict)  # True 
print('white' in dog_dict)  # False
```

方法：max/min(字典.values())

作用：获得字典中value的最大值/最小值

方法：max/min(字典，key=字典.get)

作用：获得字典中value的最大值/最小值所对应的键

```
prices = {
    'apple': 191,
    'goog': 1186,
    'ibm': 149,
    'orcl': 48,
    'acn': 166,
    'fb': 208,
    'symc': 21
}

print(max(prices.values()))							          # 1186
print(max(prices, key=prices.get))					      # goog
print(min(prices.values()))							          # 21
print(min(prices, key=prices.get))					      # symc

print(max(zip(prices.values(), prices.keys())))		# (1186,'goog')
print(min(zip(prices.values(), prices.keys())))		# (21, 'symc')
```

##### 增加

方法：字典[key] = value

作用：通过给字典中**不存在的key**进行赋值，形成新的键值对，添加到字典中。

```
person = {'name': '路飞', 'age': 17, 'face': 90}
person['height'] = 1.8
print(person)  			# {'name': '路飞', 'age': 17, 'face': 90, 'height': 1.8}
```

##### 修改

方法：字典[key] = value

作用：通过给字典中**存在的key**进行赋值，修改字典中的键值对的value。

```
person = {'name': '路飞', 'age': 17, 'face': 90}
person['age'] = 18
print(person)  			# {'name': '路飞', 'age': 18, 'face': 90}
```

##### 删除

方法：del 字典[key]

作用：删除字典中指定的键值对（删除key时候，value也就被删除了）

**注意：删除字典中不存在的key，会报错。**

**注意：del 字典（这是删除整个字典）**

```
person = {'name': '路飞', 'age': 17, 'face': 90}
del person['face']
print(person)  		# {'name': '路飞', 'age': 17}

# del person['aaa']  # 报错 KeyError
```

方法：字典.pop(key)

作用：**取出**键值对对应的值

```
person = {'name': '路飞', 'age': 18, 'face': 90}
age = person.pop('age')
print(person, age)  	# {'name': '路飞', 'height': 1.8} 18
```

### 计数器

```
a = ['apple', 'banana', 'apple', 'tomato', 'apple', 'banana']
b = {}
for i in a:
    b[i] = b.get(i, 0) + 1

print(b)    # {'apple': 3, 'banana': 2, 'tomato': 1}
```
