# (Json)文件读写、文件(夹)操作

### 文件读写

##### 文件类型

文件类型大体分为两类：

1. **文本文件**：**存储有效字符信息的文件**。
2. **二进制文件**：**音频文件、视频文件、图片文件等**

##### 读写流程

**文件读写操作流程**：打开文件 -> 操作文件（读/写） -> 关闭文件

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
'r': 读操作（默认，读出来是字符串）
'w': 写操作（清空原有数据，将文本数据写入文件）
'a': 写操作（保留以前的数据，将新的数据写入文件）
'r+': 可读、可写，文件不存在会报错，写操作时会覆盖
'w+': 可读、可写，文件不存在先创建，会覆盖
'a+': 可读、可写，文件不存在先创建，不会覆盖，追加在末尾
二进制文件：
'rb'/'br'  - 读操作（读出来的数据是二进制）
'wb'/'bw'  - 写操作（清空原有数据，将二进制数据写入文件）
注意：以读的方式'r'或'rb'打开文件，如果这个文件不存在，会报错；以写的方式'w'或'wb'打开文件，如果这个文件不存在，就会创建这个文件

编码方式 ————> 针对文本文件的读写，二进制文件读写不设置
'utf-8'、'gbk'、'gb2312'...
'''
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

##### 关闭文件

```python
#  ‘文件’指的是打开文件后赋值的变量
文件.close()
```

##### 关键字 `with`

关键字 `with`：在文件**操作结束后会自动去关闭文件**。

```python
with open() as 文件变量名：
     文件操作
```

`with open` 操作文件：

```python
with open('./lew.jpg', 'rb')as f:  # rb读二进制文件
    jpg_date = f.read()

with open('./lewr.jpg', 'wb')as f: # wb写二进制文件
    f.write(jpg_date)

with open('./lew.jpg', 'r', encoding='utf-8')as f:  # r读文本文件，以utf-8编码方式
    jpg_date = f.read()

# 写操作
with open('./lewr.jpg', 'w', encoding='utf-8')as f: # w写文本文件，以utf-8编码方式
    f.write(jpg_date)
```

##### 字符与文本

当字符编码和文本文档的编码不相对应时，打开文本文件就会出现乱码。通过设置不同编码的“写操作”来生成文本文档，查看对应关系：

| encoding的编码 | 生成文本文档的编码 |
| -------------- | ------------------ |
| ansi           | ANSI               |
| gb2312         | ANSI               |
| gbk            | ANSI               |
| unicode-escape | UTF-8              |
| utf-8          | UTF-8              |
| utf-16         | UTF-16 LE          |

##### BOM影响

之前**文本文档的编码**中由提到过，UTF-8编码有两种：UTF-8、带BOM的UTF-8，下面来看看BOM在Python程序中读取的影响：

```python
# 读取编码为UTF-8的文件
with open("Task.txt", "r",encoding='utf-8')as f:
    nr = f.read()
    print(nr, f'字符长度：{len(nr)}')              # 新华社 字符长度：3

with open("Task.txt", "r", encoding='utf-8')as f:
    nr = f.readlines()
    print(nr)                                    # ['新华社']
    print(nr[0], f'字符长度：{len(nr[0])}')        # 新华社 字符长度：3
    print(nr[0].encode())                        # b'\xe6\x96\xb0\xe5\x8d\x8e\xe7\xa4\xbe'
    
# 读取编码为带BOM的UTF-8的文件
with open("Task.txt", "r",encoding='utf-8')as f:
    nr = f.read()
    print(nr, f'字符长度：{len(nr)}')              # ﻿新华社 字符长度：4

with open("Task.txt", "r", encoding='utf-8')as f:
    nr = f.readlines()
    print(nr)                                    # ['\ufeff新华社']
    print(nr[0], f'字符长度：{len(nr[0])}')        # ﻿新华社 字符长度：4
    print(nr[0].encode())                        # b'\xef\xbb\xbf\xe6\x96\xb0\xe5\x8d\x8e\xe7\xa4\xbe'

# 注释：可以看到带BOM的UTF-8比UTF-8多了一个﻿，在列表中内容为\ufeff，编码中内容为\xef\xbb\xbf，当中ef bb bf就恰好对应了带BOM的UTF-8开头的字节流EF BB BF。
```

去掉BOM有两种方法：

1. 打开文本文件，将其另存为**不带有BOM的UTF-8**。（最好是在专业版Windows系统中能显示有两个UTF-8编码选项下操作）
2. 将字符进行 `utf-8` 编码，再以 `utf-8-sig` 解码。

```python
# 读取编码为带BOM的UTF-8的文件
with open("Task.txt", "r",encoding='utf-8')as f:
    nr = f.readlines()
    nr = nr[0]
    print(nr, f'字符长度：{len(nr)}')              # ﻿新华社 字符长度：4

with open("Task.txt", "r", encoding='utf-8')as f:
    nr = f.readlines()
    nr = nr[0].encode().decode('utf-8-sig')      
    print(nr, f'字符长度：{len(nr)}')              # 新华社 字符长度：3
```

##### 大文件读写

如果要复制的图片文件很大，一次将文件内容直接读入内存中可能会造成非常大的内存开销，为了减少对内存的占用，可以为`read`方法传入`size`参数来指定每次读取的字节数，通过循环读取和写入的方式来完成上面的操作：

```python
try:
    with open('guido.jpg', 'rb') as file1, \
        open('吉多.jpg', 'wb') as file2:
        data = file1.read(512)
        while data:
            file2.write(data)
            data = file1.read()
except FileNotFoundError:
    print('指定的文件无法打开.')
except IOError:
    print('读写文件时出现错误.')
print('程序执行结束.')
```

### Json文件的读写

##### json文件

**数据本地化：将数据保存到本地文件中，而Json文件就常被用于保存数据。**

**json文件**：以 `.json` 为后缀的**文本文件**，`json` 意思是 `javascript` 对象表示法，类似于 `javascript` 对象的一种数据格式。

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

# 将l列表，进行json格式化编码
encoded_json = json.dumps(l)                        
print(encoded_json, type(encoded_json))

# indent参数可以按照json样式可视化显示
encoded_json1 = json.dumps(l, indent=1)              
print(encoded_json1, type(encoded_json1))
'''
输出：
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

### shutil模块

`shutil` 是Python的一个高级的文件，文件夹，压缩包处理模块，拥有许多文件（夹）操作的功能，包括复制、移动、重命名、删除等等。

##### 导入模块

```python
# 使用前，需导入该模块
import shutil
```

##### 拷贝内容

```python
# copyfileobj通过打开的文件对象，将文件1的数据复制给文件2
# 以“r读方式”打开文件1.txt
with open("1.txt",'r',encoding="utf-8") as f1:
    # 以“w写方式”打开文件2.txt，将1.txt内容以覆盖形式拷贝到2.txt中
    # 以“a加方式”打开文件2.txt，将1.txt内容以追加形式拷贝到2.txt中
    with open("2.txt","w",encoding="utf-8") as f2:
        shutil.copyfileobj(f1, f2)

# copyfile直接通过文件名，将1.txt内容以覆盖形式拷贝到2.txt中
shutil.copyfile("1.txt", "2.txt")

# copymode仅拷贝1.txt权限给2.txt，但内容、组、用户均不变
shutil.copymode("1.txt", "2.txt")

# copystat拷贝1.txt状态的信息给2.txt，包括：mode、bits、atime、mtime、flags
shutil.copystat("1.txt", "2.txt")
```

##### 拷贝文件(夹)

```python
# copy（文件及路径，目标路径）：复制文件（目标文件要是不存在会自动创建）
# 当前目录下的文件，拷贝到目标路径下
shutil.copy('文件名.后缀名', '目标路径')
# 非当前目录下的文件，拷贝到目标路径下
shutil.copy('路径\文件名.后缀名', '目标路径')

# copy2（文件及路径，目标路径）：复制文件，保留原有文件的信息（操作时间和权限等）
# 当前目录下的文件，拷贝到目标路径下
shutil.copy2('文件名.后缀名', '目标路径')
# 非当前目录下的文件，拷贝到目标路径下
shutil.copy2('路径\文件名.后缀名', '目标路径')

# copytree（来源路径，目标路径）：拷贝整个文件夹
shutil.copytree('来源路径', '目标路径')
```

##### 移动文件(夹)

```python
# move：递归移动文件或者文件夹（慎用）
# 当前目录下的文件，移动到目标路径下
shutil.move('文件名.后缀名', '目标路径')
# 非当前目录下的文件，移动到目标路径下
shutil.move('路径\文件名.后缀名', '目标路径')
# 将文件夹移动到目标路径下
shutil.move('路径\文件夹', '目标路径')
```

##### 删除文件(夹)

```python
# rmtree递归删除文件或者文件夹，它不是原子操作（如同rm -rf一样危险，慎用）
# 删除当前目录下的文件
shutil.rmtree('文件名.后缀名')
# 删除非当前目录下的文件
shutil.rmtree('路径\文件名.后缀名')
# 删除文件夹
shutil.rmtree('路径\文件夹')
```

##### 压缩文件

```python
# make_archive方法创建压缩包并返回文件路径
'''
参数详解：
base_name：文件名时，则保存至当前目录；路径时，则保存至指定路径
如：data_bak=>保存至当前路径
如：/tmp/data_bak=>保存至/tmp/
format：压缩包种类，“zip”, “tar”, “bztar”，“gztar”
root_dir：要压缩的文件夹路径（默认当前目录）
owner：用户，默认当前用户
group：组，默认当前组
logger：用于记录日志，通常是logging.Logger对象
'''
# 将/data下的文件打包放置当前程序目录
ret = shutil.make_archive("data_bak", 'gztar', root_dir='/data')  
# 将/data下的文件打包放置/tmp/目录
ret = shutil.make_archive("/tmp/data_bak", 'gztar', root_dir='/data')
```

##### 查看磁盘使用量

```python
# disk_usage（盘符）：查看磁盘使用量（参数如果是路径，那么结果仍然是路径所在盘符的使用量）
res=shutil.disk_usage('C:')
print(res)
'''
输出：
usage(total=120031539200, used=78561628160, free=41469911040)
'''
```

