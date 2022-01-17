# reids-py库

在执行下面操作时，确保电脑中安装了Redis，并且已经在本机的6379端口启动了Redis服务。

## 安装连接

### 安装redis-py

redis-py 库提供两个类 Redis 和 StrictRedis 来实现 Redis 的命令操作。

**StrictRedis 实现了绝大部分官方的命令，参数一一对应(官方推荐)。**

**Redis是 StrictRedis 的子类，它的主要功能是用于向后兼容旧版本库里的几个方法。**

安装要求：Python3.6+

安装命令：`pip install redis-py`

?> redis-py 的API保留了Redis API的原始风格，所以使用起来不会有不习惯的感觉。

### 连接Redis

```python
# 导入redis
from redis import StrictRedis

# StrictRedis对象，默认传入地址localhost、端口6379、选择0号数据库（不写默认0号）、密码为None。
client = StrictRedis(host='localhost', port=6379, db='0', password='None')
```

## 数据操作

### 常用数据类型

```python
#1.字符串string 
client.set("hello","world")  # True 
print(client.get("hello"))   # world 

#2.哈希hash 
client.hset("myhash","f1","v1") 
client.hset("myhash","f2","v2") 
print(client.hgetall("myhash"))  # {'f1': 'v1', 'f2': 'v2'} 

#3.列表list 
client.rpush("mylist","1") 
client.rpush("mylist","2") 
client.rpush("mylist","3") 
print(client.lrange("mylist", 0, -1))  # ['1', '2', '3']

#4.集合set 
client.sadd("myset","a") 
client.sadd("myset","b") 
client.sadd("myset","a")         
print(client.smembers("myset"))  # set(['a', 'b']) 

#5.有序集合zset 
client.zadd("myzset","99","tom") 
client.zadd("myzset","66","peter") 
client.zadd("myzset","33","james") 
print(client.zrange("myzset", 0, -1, withscores=True))  # [('james', 33.0), ('peter', 66.0), ('tom', 99.0)] 
```

### Pipeline管道

**redis中client（客户端）与server（服务端）之间采用的是请求应答的模式：**

```
Client: command1
Server: response1
Client: command2
Server: response2
…
```

在这种情况下，如果要完成10个命令，则需要20次交互才能完成。因此，即使redis处理能力很强，仍然会受到网络传输影响，导致吞吐量上不去。而在管道（pipeline）模式下，多个请求可以变成只需要2次交互。这样网络传输上能够更加高效，加上redis本身强劲的处理能力，给数据处理带来极大的性能提升。

```
Client: command1，command2… 
Server: response1，response2…
```

redis-py支持Redis的Pipeline功能，大家知道redis提供了 `mset`、`mget`方法，但没有提供 `mdel` 方法，如果想实现，可以借助pipeline实现：

```python
import redis
# 生成连接
client = redis.StrictRedis(host='localhost', port=6379)
# 生成Pipeline，参数transaction等于False代表不使用事务
pipeline = client.pipeline(transaction=False)

def mdel(keys):
    for key in keys:
        # 将删除命令封装到Pipeline中，此时命令并没有真正执行
        print(pipeline.delete(key))
    # 执行Pipeline
    return pipeline.execute()
```

### Lua脚本

Lua语言是在1993年由巴西一个大学研究小组发明，其设计目标是作为嵌入式程序移植到其他应用程序，**它是由C语言实现的**，虽然简单小巧但是功能强大，所以许多应用都选用它作为脚本语言，尤其是在游戏领域，例如大名鼎鼎的暴雪公司将Lua语言引入到“魔兽世界”这款游戏中，Rovio公司将Lua语言作为“愤怒的小鸟”这款火爆游戏的关卡升级引擎，Web服务器Nginx将Lua语言作为扩展，增强自身功能。**Redis将Lua作为脚本语言可帮助开发者定制自己的Redis命令。**

`redis-py` 中执行Lua脚本和redis-cli十分类似，它提供了三个重要的函数实现Lua脚本的执行：

```
eval(String script, int keyCount, String... params) 
```

- script：Lua脚本内容。 

- keyCount：键的个数。 

- params：相关参数KEYS和ARGV。

在 `redis-py` 中执行，方法如下：

```python
import redis 
client = redis.StrictRedis(host='127.0.0.1', port=6379)
# 简单的Lua脚本
script = "return redis.call('get',KEYS[1])" 
#输出结果为world 
print(client.eval(script,1,"hello"))
```

**script_load和evalsha函数要一起使用**

```
script_load(String script) 
evalsha(scriptSha, keyCount, params：
```

- scriptSha：脚本的SHA1。 

- keyCount：键的个数。 

- params：相关参数KEYS和ARGV。

完整代码如下：

```python
import redis
client = redis.StrictRedis(host='127.0.0.1', port=6379)
script = "return redis.call('get',KEYS[1])"
# script_load将脚本加载到Redis中
scriptSha = client.script_load(script)
print (client.evalsha(scriptSha, 1, "hello"))
```
