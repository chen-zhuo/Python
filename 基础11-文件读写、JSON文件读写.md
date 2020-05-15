# 文件读写、JSON文件读写

### 文件读写

##### 文件类型及读写流程

文件类型一般分为两类：

1. **文本文件**：**存储有效字符信息的文件**。
2. **二进制文件**：**音频文件、视频文件、图片文件等**

**读写操作流程**：打开文件 -> 操作文件（读/写） -> 关闭文件

##### 打开文件

```python
open(文件路径，打开方式，编码方式)

'''
文件路径 ————> 决定打开哪个文件
绝对路径：文件在磁盘中的完整位置，一般放在工程外面的文件使用绝对路径
# C:\酌\Desktop\新建文本文档.txt
相对路径：相对于当前进行操作的文件的位置./或../，一般文件在工程目录下的某个位置使用相对路径（推荐）
# ./test1.txt   # 表示test1文件和操作文件在同一文件夹下
# ../test1.txt  # 表示test1文件和操作文件在不同一文件夹下

打开方式 ————> 进行什么样的操作
文本文件：
'r'(默认): 读操作（读出来是字符串）
'w': 写操作（清空原有数据，将文本数据写入文件）
'a': 写操作（保留以前的数据，将新的数据写入文件）
r+ : 可读、可写，文件不存在会报错，写操作时会覆盖
w+ : 可读，可写，文件不存在先创建，会覆盖
a+ : 可读、可写，文件不存在先创建，不会覆盖，追加在末尾
二进制文件：
'rb'/'br'  - 读操作（读出来的数据是二进制）
'wb'/'bw'  - 写操作（清空原有数据，将二进制数据写入文件）
注意：以读的方式'r'或'rb'打开文件，如果这个文件不存在，会报错；以写的方式'w'或'wb'打开文件，如果这个文件不存在，就会创建这个文件

编码方式 ————> 针对文本文件的读写，二进制文件读写不设置
encoding='utf-8'
'''
```

##### 关闭文件

```python
#  ‘文件’指的是打开文件后赋值的变量
文件.close()
```

##### 文件读操作

```python
文件.read(n)	- n：设置读的长度，可以不写
```

文本文件读操作：

```python
# 同一目录下新建test1.txt文件，内容如下：
'''
疑是地上霜
'''

f = open('./test1.txt', 'r', encoding='utf-8')  # 打开方式'r'，并设置编码格式utf-8
content1 = f.read()       # 一次性读取文本中全部的内容(不包括换行符)，以字符串的形式返回结果
print(content1)           # 疑是地上霜

content2 = f.readline()   # 只读取文本第一行的内容，以字符串的形式返回结果
print(content2)           # 疑是地上霜

content3 = f.readlines()  # 一次性读取文本中全部的内容(包括换行符)，以列表的形式返回结果，每行信息就是一个列表的参数
print(content3)           # ['疑是地上霜']

f.close()
```

二进制文件读操作：

```python
f = open('./img.jpg', 'rb')  # 打开方式'rb'，不要设置编码格式
image_date = f.read()
print(image_date)            # b'\xff\xd8\xff\xe0\...（b开头的二进制流数据）

f.close()
```

##### 文件写操作

```python
文件.write('写入的内容')
```

文本文件写操作：

```python
# 同一目录下新建test1.txt空文件

f = open('./test1.txt', 'w', encoding='utf-8')  # 打开方式'w'，并设置编码格式utf-8
f.write('疑是地上霜')   # 向文件里面写入'疑是地上霜'
f.close()

# 打开test1.txt文件
'''
疑是地上霜
'''
```

二进制文件写操作：

```python
t = open('./img.jpg', 'rb')  # 打开有图像的图片img.jpg，打开方式'rb'，不要设置编码格式
image_date = t.read()        # 读取图片的二进制数据

f = open('./lew.jpg', 'wb')  # 打开无图像的图片lew.jpg，打开方式'wb'
f.write(image_date)	         # 将上面的图片img.jpg的二进制数据写入图片lew.jpg当中
f.close()
```

##### 关键字 `with`

**关键字 `with`**：在文件**操作结束后会自动去关闭文件**。

```python
with open() as 文件变量名：
     文件操作
```

`with open` 操作文件：

```python
# 读操作
with open('./lew.jpg', 'rb')as f:  # rb读二进制文件
    jpg_date = f.read()

# 写操作
with open('./lewr.jpg', 'wb')as f: # wb写二进制文件
    f.write(jpg_date)
```

### `JSON` 文件的读写

##### `json` 文件

**数据本地化**：**将数据保存到本地文件中**，方式有：文本文件、`json` 文件、数据库...

`json` ：单词的意思是 `javascript` 对象表示法，类似于 `javascript` 对象的一种数据格式。

`json` 文件：以 `.json` 为后缀的**文本文件**。

1. `json` 文件里面的数据内容是 `json` 格式的，是一种与语言无关的数据交换的格式。
2. `json` 有两种格式，**对象格式**：`{"key1":obj,"key2":obj...}`，**数组/集合格式**：`[obj,obj,obj...]`。
3. 支持的数据类型：字符串、数字、布尔值、列表、字典。
4. **`json` 格式中的字符串全为双引号，没有单引号**。

`Python` 中有内置的 `json` 模块，专门处理 `json` 格式数据的，直接导入就可用。

##### `json` 转 `Python` 

`json.loads()`：将 `json` 格式解码成 `Python` 数据类型。

```python
# 导入json模块
import json                                          

encoded_json = ["iplaypython", [1, 2, 3], {"name": "xiaoming"}]
# 将 json 格式解码成 Python 数据类型
analys_json = json.loads(encoded_json)  
print(analys_json, type(analys_json))

'''
输出：
['iplaypython', [1, 2, 3], {'name': 'xiaoming'}] <class 'list'>
'''
```

##### `Python` 转 `json` 

`json.dumps()`：将 `Python` 数据类型编码为 `json` 格式。

```python
# 导入json模块
import json                                          

# 创建一个l列表
l = ['iplaypython', [1, 2, 3], {'name': 'xiaoming'}]
print(l, type(l))

# 将l列表，进行json格式化编码
encoded_json = json.dumps(l)                        
print(encoded_json, type(encoded_json))

# 将l列表，进行json格式化编码，并按照json样式可视化显示
encoded_json1 = json.dumps(l, indent=1)              
print(encoded_json1, type(encoded_json1))
'''
输出：
['iplaypython', [1, 2, 3], {'name': 'xiaoming'}] <class 'list'>
# 注释：将一个list列表对象，进行了json格式的编码，单引号变双引号。
["iplaypython", [1, 2, 3], {"name": "xiaoming"}] <class 'str'>
# 按照json样式可视化显示
[
 "iplaypython",
 [
  1,
  2,
  3
 ],
 {
  "name": "xiaoming"
 }
]<class 'str'>
'''
```

##### `json` 文件读操作

`json.load()`：**将文件中的 `json` 数据读取出来**。

```python
import json

f = open('./tt.json','r')
hehe = json.load(f)
print(hehe)

'''
输出：
[{"a": "aaa", "b": "bbb", "c": [1, 2, 3, [4, 5, 6]]}, 33, "tantengvip", true]
'''
```

##### `json` 文件写操作

`json.dump(x,f)`：`x` 是数据对象，`f` 是文件对象，将 `json` 字符串写入到文本文件中。

```python
import json

data = [{"a": "aaa", "b": [1, 2, (4, 5)]}, 33, 'tantengvip', True]

data2 = json.dumps(data)  # 进行json格式转码
print(data2)  # [{"a": "aaa", "b": [1, 2, [4, 5]]}, 33, "tantengvip", true]

f = open('./tt.json','a') # 将新文件对象和操作类型赋值给f（a代表追加写入）
json.dump(data2,f)        # 将data2的JSON数据写入到f中

'''
结果：生成了一个tt.json文件，保存了json格式的数据。
"[{\"a\": \"aaa\", \"b\": \"bbb\", \"c\": [1, 2, 3, [4, 5, 6]]}, 33, \"tantengvip\", true]"
'''
```

