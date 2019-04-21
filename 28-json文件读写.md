# JSON文件的读写

### 了解json文件

##### 数据本地化

数据本地化：**将数据保存到本地文件中(文本文件、json文件、数据库)**

##### json文件

json文件：以 **.json 为后缀**，内容是**json格式的文本文件**。

##### json格式

1. 最外层只能是**一个字典**、**一个列表**、**一个字符串**
2. **最外层是字典，字典里面就必须是键值对**
3. **最外层是列表，数组里面的内容必须是数组数据**
4. 支持的数据类型：字符串、数字、布尔值、列表、字典、空(None)

**注意：json格式中的字符串全为双引号，没有单引号。**

##### json模块

Python中有内置的json模块，专门处理json数据的，直接导入就可用。

```
import json
```

### 编码转换

##### json.dumps()

**将Python数据类型编码为json格式**

```
# 导入json模块
import json

# 创建一个l列表
l = ['iplaypython',[1,2,3], {'name':'xiaoming'}] 

# 将l列表，进行json格式化编码
encoded_json = json.dumps(l) 

print(type(encoded_json))
print(encoded_json)

结果：
<class 'str'>
["iplaypython", [1, 2, 3], {"name": "xiaoming"}]

# 注释：将一个list列表对象，进行了json格式的编码，单引号变双引号。
```

##### json.loads()

**将json格式解码成Python数据风格**

```
# 将上面的json格式encoded_json进行解析
analys_json = json.loads(encoded_json)
print(type(analys_json))
print(analys_json)

结果：
<class 'list'>
['iplaypython', [1, 2, 3], {'name': 'xiaoming'}]
```

### json读写

##### json.dump()

**json.dump(x,f)，x是数据对象，f是文件对象，将json字符串写入到文本文件中。**

```
import json

data = [{"a":"aaa","b":[1,2,(4,5)]},33,'tantengvip',True]

# 进行json格式转码
data2 = json.dumps(data)
print(data2)  # [{"a": "aaa", "b": [1, 2, [4, 5]]}, 33, "tantengvip", true]

# 将新文件对象和操作类型赋值给f（a代表追加写入）
f = open('./tt.txt','a')

# 将data2的JSON数据写入到f中
json.dump(data2,f)


结果：生成了一个tt.txt文件，保存了json格式的数据。
"[{\"a\": \"aaa\", \"b\": \"bbb\", \"c\": [1, 2, 3, [4, 5, 6]]}, 33, \"tantengvip\", true]"
```

##### json.load()

**将文件中的json数据读取出来**

```
f = open('./tt.txt','r')
hehe = json.load(f)
print(hehe)

结果：
[{"a": "aaa", "b": "bbb", "c": [1, 2, 3, [4, 5, 6]]}, 33, "tantengvip", true]
```

如果你要处理的是文件而不是字符串，你可以使用json.dump()和json.load()来编码和解码JSON数据。

```
# Writing JSON data
with open('data.json', 'w') as f:
    json.dump(data, f)

# Reading data back
with open('data.json', 'r') as f:
    data = json.load(f)
```
