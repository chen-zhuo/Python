# Queue队列

为了进一步学习爬虫，这里我们需要掌握一个东西：**队列（Queue）**

这里我们将Queue队列分为两种：**消息队列（Message Queue，简称MQ）、任务队列（Task Queue，简称TQ）**

## 消息队列MQ

前面我们学习了异步爬虫，在很大程度上提升了采集的效率，其“策略”就是**将所有要爬取的链接先获取下来，用列表存储，再启动爬虫程序，不重复的获取链接，省掉了其他爬虫等待链接的时间和本身的闲置时间，因此能成倍提升采集效率。**这里存储链接的列表，就可以称之为**消息队列**。

消息队列：**简称MQ，是一种应用间的通信方式，消息发送后可以立即返回，由消息系统来确保消息的可靠传递。**

消息队列特点：**阅后即焚**，即消息队列里面的消息被使用后就被销毁了。

消息队列作用：在高并发环境下，由于来不及同步处理，请求往往会发生堵塞，**通过使用消息队列，我们可以异步处理请求，从而缓解系统的压力**。

### 生产消费模型

生产者(producer)：**发送消息的程序。**

![20170524102259570](image/20170524102259570.png)

队列(queue)：**存储消息的队列。** 一个队列只受到主机的内存和磁盘的限制，它实际上是个大的消息缓冲区。

![20170524104231217](image/20170524104231217.png)

消费者(consumer)：**等待接收消息的程序**。

![20170524104158185](image/20170524104158185.png)

消息流：**生产者只管创建消息并发布到队列中，消费者只管监听或使用队列中的消息。**

![1436447-20181121135142706-7726795](image/1436447-20181121135142706-7726795.png)

?> 提示：多个生产者可以发送消息到一个队列，多个消费者可以监听或使用同一个队列中的消息。

### Queue消息队列

python中有支持队列的两个标准库：进程队列模块 `multiprocessing`、线程队列模块 `queue`（两个模块的用法一模一样）

进程队列模块 `multiprocessing`：**用于多进程，它通过管道以及锁和信号量机制来协调多个进程，数据可以在多进程之间共享同一个变量的数据。**

```python
import multiprocessing

# 创建一个进程队列
q = multiprocessing.Queue()
```

线程队列模块 `queue`：**用于多线程，它可以指定缓冲区大小的阻塞队列，但数据无法在多个进程之间共享。**

```python
import queue

# 创建线程队列，Queue先进先出模式
q1 = queue.Queue()

# empty()判断队列是否为空
print(q1.empty())	# True
# 获取队列q的长度
print(q1.qsize())	# 0

for i in range(3):
	# 向队列q中插入变量数据i
    q1.put(i)

# empty()判断队列是否为空
print(q1.empty())	# False
# 获取队列q的长度
print(q1.qsize())	# 3

# get()获取数据，每次get只能取一个数据
print(q1.get())  # 0
print(q1.get())  # 1
print(q1.get())  # 2


# 创建线程队列，LifoQueue先进后出模式
# maxsize参数表示队列中能存放的数据个数的上限；默认等于0，表示队列大小没有限制。
q2 = queue.LifoQueue(maxsize=4)

# 插入数据0，1，2，3后，消息队列数据达到上限，发生阻塞，不会执行下面的print语句。
for i in range(5):
    q2.put(i)	
print('程序结束')

# 队列q2中一边插入数据，一边消费数据，所以不会发生阻塞，可以执行下面的print语句。
for i in range(5):
    q2.put(i)
    print(q2.get())	
print('程序结束')	
'''
输出：4 3 2 1 0 程序结束
解释：put(i)插入一次数据，get()获取一次数据，数据量不会达到队列上限4，能执行print语句。
'''
```

?> 提示：`queue` 和 `multiprocessing` 模块，两者 `Queue` 队列的 `get`、`put`、`empty`、`qsize` 等方法名称是相同的。

### Redis消息队列

**Redis是时下比较火的内存数据库，以简单高效著称，具有丰富的数据类型，常用于缓存数据，也支持队列queue，我们可以将Redis作为message broker（消息中间人）。**

Redis目前只能实现简单的消息队列功能，它所具有的优点：

- 极低延迟：得益于Redis本身的高效，消息可以极低延迟传递。
- 上手容易：只需组合使用两三个命令，即可完成消息传递。
- 查看便利：可以便利地查看队列内容；利用 `MONITOR` 命令甚至可以实时监控消息入队和出队。
- 单线程锁：**Redis本身是一个单线程的，相当于所有操作都有一把天然的排他锁，因此我们不用担心在并发下同一条消息会传递给多个使用者**。

#### 可靠消息队列

可靠消息队列：**由两个列表组成，一个存储待处理（`pending`）的消息，另一个存储处理中（`processing`）的消息。**

可靠消息队列运行流程：

1. **生产者将所有任务添加至待处理列表里面；**
2. **消费者从待处理列表里面获取任务，并将该任务添加至处理中列表；**
3. **消费者成功处理任务后，再将处理中列表里面的任务删除。**

使用可靠消息队列的优势在于，可以防止处理异常，而导致任务丢失。详情请阅读：《DataBase》中的《Redis-消息队列》

#### 启动配置服务

首先确保本机已经安装了Redis，接着修改Redis的配置文件`redis.windows.conf`，方便使用：

1. 注释绑定，在 `bind 127.0.0.1` 前面加上一个注释，允许其他IP能连接Redis。

![QQ截图20200917224556](image/QQ截图20200917224556.png)

2. 设置密码，取消 `requirepass` 前面的注释，在后面加上自己的密码 。

![QQ截图20200917223824](image/QQ截图20200917223824.png)

3. 配置启动，为了让修改的配置起作用，在命令行通过配置文件来启动Redis。

```
redis-server "Redis安装路径\redis.windows.conf"
```

![QQ截图20200917225122](image/QQ截图20200917225122.png)

4. 连接Reids，再启一个命令行，通过密码来连接Redis进行操作。

```
方法一：连接时，通过密码进行身份验证
redis-cli -a 密码

方法二：连接后，通过密码进行身份验证
redis-cli
auth 密码
```

![QQ截图20200917230443](image/QQ截图20200917230443.png)

#### 任务处理

**添加任务**：向 Redis 里面添加需要处理的任务：

```python
# 导入Redis模块
from redis import StrictRedis

# 连接局域网内IP为192.168.0.3上密码为123456的Redis的0号数据库
client = StrictRedis(host='192.168.0.3', port=6379, db='0', password='123456')

# 循环添加任务
for message in ['任务1', '任务2', '任务3']:    
    # 通过lpush命令将任务添加到pending待处理列表左侧    
    print(client.lpush("pending", message))
```

**获取任务**：弹出 `pending` 待处理列表最右边的任务，并且将任务添加到处理中的 `processing` 任务列表的最左边。

```python
# 弹出pending待处理列表最右边的任务，并且将任务添加到处理中的processing任务列表中
message = client.rpoplpush('pending', 'processing')
```

**删除任务**：当任务成功处理完毕，再将任务从处理中的 `processing` 任务列表删除。

```python
# 删除处理中的processing任务列表中从左至右与message内容相同的第一条任务
client.lrem('processing', 1, message)
```

### RabbitMQ框架

**RabbitMQ是一个使用Erlang语言开发、基于AMQP协议（高级消息队列协议）实现专门做队列的框架，其主要特点有面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全性。在队列方面要比redies队列性能要好，支持的功能会更多，消息的可靠性更强，且支持多种语言: Java、.NET、PHP、Python、JavaScript、Ruby、Go等。**

#### 模型介绍

既然RabbitMQ也是一种队列，那和前面说的线程queue和进程queue有什么区别呢？

- **线程queue只能在同一个进程下进行数据交互；**
- **进程queue只能在父进程和子进程之间，或者同一父进程下的子进程之间做数据交互。**

但如果需要对不同进程（eg：微信和qq）两个独立的程序间通信，那么有如下三种解决方法：

- 方法1：就是直接把数据写在硬盘（disk）上然后各自的进程读取数据就可以，但是由于硬盘的读写速度太慢，效率太低；
- 方法2：自己写个socket，直接做数据交互，问题是如果改变程序，或者再加一个程序，需要对写好的socket进行修改，还要处理黏包什么的复杂的连接关系，维护成本太高。
- 方法3：**利用已有的中间商（代理），这个broker其实就是封装好的socket，我们拿来直接用就好了。这里的broker，就有RabbitMQ，ZeroMQ，ActiveMQ等。**

![QQ截图20220405175826](image/QQ截图20220405175826.png)

**RabbitMQ是一个消息中间人（broker）: 它接收、存储并且发送消息。**

可以想象一个邮局：当你把想要寄出的信放到邮筒里时，邮递员会把信件送到收信人那里。 在这个比喻中, RabbitMQ 就是一个邮筒（接收）, 同时也是邮局（存储）和邮递员（转发） 。和邮局的主要不同点在于RabbitMQ不处理纸质信件, 而是 接收（accepts）, 存储（stores） 和转发（forwards）二进制数据块 —— 消息（messages）。

#### 安装RabbitMQ

以下资源在官网下载太慢，这里介绍另外一个资源下载地址：https://sourceforge.net/，直接在搜索栏输入需要的资源下载即可。

安装我们首先看看[官网](https://www.rabbitmq.com/install-windows.html)声明的依赖项：

![QQ截图20220405173833](image/QQ截图20220405173833.png)

因为RabbitMQ使用Erlang语言开发，因此首先我们需要安装一个Erlang的语言环境。这里我们下载的版本为：Erlang/OTP 24.3.3。**下载好了我们进行安装，注意要按照官网说明的“以管理员身份运行”安装：**

![QQ截图20220405184348](image/QQ截图20220405184348.png)

安装好后，我们将Erlang安装路径添加到系统变量当中，名称为ERLANG_HOME：

![QQ截图20220405184612](image/QQ截图20220405184612.png)

接着继续修改Path的值，添加Erlang安装路径下的bin路径：

![QQ截图20220405184838](image/QQ截图20220405184838.png)

保存修改，我们打开命令行，执行 `erl` 命令，出现下图说明安装成功：

![QQ截图20220405185117](image/QQ截图20220405185117.png)

接下来我们安装RabbitMQ，其版本为3.9.14版本，同样“以管理员身份运行”安装：

![QQ截图20220405185441](image/QQ截图20220405185441.png)

安装好后，我们将RabbitMQ安装路径添加到系统变量当中，名称为RABBITMQ_SERVER：

![QQ截图20220405185857](image/QQ截图20220405185857.png)

接着继续修改Path的值，添加RabbitMQ安装路径下的sbin路径：

![QQ截图20220405190014](image/QQ截图20220405190014.png)

保存后，在重启命令行加载新的信息，执行 `rabbitmq-plugins.bat enable rabbitmq_management` 命令安装插件，出现下面信息说明安装成功：

![QQ截图20220405190623](image/QQ截图20220405190623.png)

这时我们再去访问本机的15672端口：http://localhost:15672/，出现如下页面：

![QQ截图20220405190952](image/QQ截图20220405190952.png)

**用户名和密码都为guest**，点击Login登录，进到如下页面，则说明配置成功了：该页面展示了一些基本的信息，例如RabbitMQ版本、Erlang版本、Nodes节点状态等。

![QQ截图20220405191722](image/QQ截图20220405191722.png)

#### 操作RabbitMQ

这里所有的用法都是基于上面RabbitMQ是工作在‘localhost’上且端口号为15672，能在浏览器里访问http://localhost:15672这个地址。

![1415597-20190306075858416-1490847372](image/1415597-20190306075858416-1490847372.png)

python操作RabbitMQ需要的模块有上述几种选择，我们用最简单的pika，用pip直接安装：

```
pip install pika
```

##### 生产者消费者

首先，我们来实现**进程间的通信，从本质上讲是一个生产者-消费者模型，即一个进程作为生产者往RabbitMQ消息队列放入消息，另一个进程作为消费者监听并处理消息队列中的消息。**

![QQ截图20220405225256](image/QQ截图20220405225256.png)

##### 消息分发

**定义一个消息队列**，执行后就会生成一个名称为scrape的队列：

```python
import pika

# 连接本地的RabbitMQ服务
connect = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
# 声明一个频道，即channel
channel = connect.channel()
# 声明一个名称为scrape的队列
channel.queue_declare(queue='scrape')
# 确认完成后，可以将连接关闭
connect.close()
```

回看之前的RabbitMQ管理页面，就会看到Queues的值变为了1：

![QQ截图20220405202749](image/QQ截图20220405202749.png)

点进去，就就可以看到该消息队列的详细信息，例如名称、类型、消息数量等：

![QQ截图20220405202920](image/QQ截图20220405202920.png)

**定义一个生产者**，执行后将消息 `b'Hello world!'` 放入队列中：

```python
import pika

connect = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connect.channel()
# exchange先赋值一个空字符串（后面讲），routing_key队列名称，body放入的消息体（byte类型数据）
channel.basic_publish(exchange='', routing_key='scrape', body='Hello world!'.encode())
connect.close()
```

在页面上也能看到，消息的总数量也变成了1，且有1个消息准备好被使用了，消息使用速率是每秒0个：

![QQ截图20220405204541](image/QQ截图20220405204541.png)

**定义一个消费者**，启动后会监听scrape队列的变动，如果有消息进入，就获取并消费，回调callback方法，打印输出结果：

```python
import pika

connect = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connect.channel()

def callback(ch, method, properties, body):
    print(f'Get {body}')  # Get b'Hello world!'

# queue='scrape'消息来源scrape队列
# auto_ack=True获取消息后，通知消息队列移除该消息；
# on_message_callback=callback消息来临时，执行回调函数callback。
channel.basic_consume(queue='scrape', auto_ack=True, on_message_callback=callback)

channel.start_consuming()
```

执行后，回看管理页面，State状态由running变为idle，而且Ready和Total的数量也变为了0个：

![QQ截图20220405224927](image/QQ截图20220405224927.png)

!> 注意：执行启动生产者或消费者前，都要保证所用的消息队列存在。

##### 优先级队列

前面我们了解的仅是队列最基本的用法，RabbitMQ还有一些高级功能，例如，**优先级队列，简单说就是生产者发送的消息具有优先级，队列会优先接收优先级高的消息。**实现优先级队列也很简单，只需要在声明队列时，添加一个 `x-max-priority` 参数来指定最大优先级即可。

**我们删除以前的重名队列，重新声明优先级队列，代码如下：**

```python
import pika

# 连接本地的RabbitMQ服务
connect = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
# 声明一个频道，即channel
channel = connect.channel()
# 声明一个名称为scrape的队列，指定队列的最大优先级为100
channel.queue_declare(queue='scrape', arguments={'x-max-priority': 100})
```

声明后，可以看到队列的Features特征多出了 `Pri` 标识，表示该队列是一个优先级队列：

![QQ截图20220406233224](image/QQ截图20220406233224.png)

生产者代码：

```python
# data, priority分别代表消息的内容和消息的优先级
for data, priority in (('world', 80), ('!', 70), ('Hello', 90)):
    channel.basic_publish(
        exchange='',
        routing_key='scrape',
        properties=pika.BasicProperties(priority=int(priority),),
        body=data.encode())
```

填充后，可以看到队列里面的数量都变为了3个：

![QQ截图20220406233224 - 副本](image/QQ截图20220406233224 - 副本.png)

消费者代码保持不变，直接运行，最后输出的如下：**可以看到内容是按优先级的大小，从高到低进行了输出。**

```python
'''
输出：
Get b'Hello'
Get b'world'
Get b'!'
'''
```

##### 队列持久化

RabbitMQ还提供了队列持久化存储，**如果不设置持久化存储，那么RabbitMQ重启后数据就没有了。**实现队列持久化也很简单，只需要在声明队列时，添加一个 `durable` 参数等于 `True` 即可。

**我们删除以前的重名队列，重新声明持久队列，代码如下：**

```python
import pika

connect = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connect.channel()
# durable=True声明持久队列
channel.queue_declare(queue='scrape', arguments={'x-max-priority': 100}, durable=True)
```

声明后，可以看到队列的Features特征变成了 `D`、`Pri` 标识，表示该队列是一个持久化的优先级队列：

![QQ截图20220406234834](image/QQ截图20220406234834.png)

这时生产者的代码改为如下：

```python
# 向持久化队列里面添加消息时，需要指定BasicProperties对象的delivery_mode为2
for data, priority in (('world', 80), ('!', 70), ('Hello', 90)):
    channel.basic_publish(
        exchange='',
        routing_key='scrape',
        properties=pika.BasicProperties(priority=int(priority), delivery_mode=2),
        body=data.encode())
```

## 任务队列TQ

**Celery 是一款 Python 编写的非常简单、灵活、可靠的分布式消息队列工具，可用于处理大量消息，处理实时数据以及任务调度，并且提供了一整套操作此系统的一系列工具。**Celery可以在一台机器上运行，也可以在多台机器上运行，甚至可以跨数据中心运行，用来提高Celery的高可用性以及横向扩展能力。

### 工作流程

一个完整的Celery工作流程当中，有四个部分：**消息生产者（producer）、消息中间件（message broker）、 任务执行单元（worker）、 任务结果存储（result store）。**

消息生产者（producer）：**提供消息的客户端。**

消息中间件（broker）：**Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。**包括，RabbitMQ，Redis，MongoDB，SQLAlchemy等，其中RabbitMQ、Redis比较稳定，其他处于测试阶段。

任务执行单元（worker）：**worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。**

任务结果存储（result store）：**存储worker执行的任务的结果，支持AMQP、Redis、MongoDB、MySQL、SQLAlchemy、Django ORM、Apache Cassandra、Elasticsearch等主流数据库。**

工作流程：**客户端向消息队列发送一条消息，然后中间人（Broker）将消息传递给某一个工人（Worker），工人（Worker）执行中间人（Broker）分配的任务，最后将执行结果进行存储（Storage）。**

![QQ截图20220329142946](Image/QQ截图20220329142946.png)

### 使用优势

Celery优势具体有下面几点：

- 简单：Celery 上手比较简单，不需要配置文件就可以直接运行。
- 高可用：如果出现丢失连接或连接失败，工人（Worker）和客户端会自动重试，并且中间人通过“主/主”、“主/从”的方式来进行提高可用性。
- 快速：单个 Celery 进行每分钟可以处理数以百万的任务，而且延迟仅为亚毫秒（使用 RabbitMQ、 librabbitmq 在优化过后）。
- 灵活：Celery 的每个部分几乎都可以自定义扩展和单独使用，例如自定义连接池、序列化方式、压缩方式、日志记录方式、任务调度、生产者、消费者、中间人（Broker）等。

Celery在并发、序列化、压缩支持如下：

- 并发：任务并发执行支持prefork、eventlet、gevent、threads的方式；

- 序列化：序列化支持pickle、json、yaml、msgpack等；
- 压缩：压缩支持zlib、bzip2。

Celery的其他功能如下：

- 监控：可以针对整个流程进行监控，内置的工具或可以实时说明当前集群的概况。
- 调度：可以通过调度功能在一段时间内指定任务的执行时间 datetime，也可以根据简单每隔一段时间进行执行重复的任务，支持分钟、小时、星期几，也支持某一天或某一年的Crontab表达式。
- 工作流：可以通过“canvas“进行组成工作流，其中包含分组、链接、分块等等。简单和复杂的工作流程可以使用一组“canvas“组成，其中包含分组、链接、分块等。
- 资源（内存）泄漏保护：--max-tasks-per-child 参数适用于可能会出现资源泄漏（例如：内存泄漏）的任务。
- 时间和速率的限制：您可以控制每秒/分钟/小时执行任务的次数，或者任务执行的最长时间，也将这些设置为默认值，针对特定的任务或程序进行定制化配置。
- 自定义组件：开发者可以定制化每一个职程（Worker）以及额外的组件。职程（Worker）是用 “bootsteps” 构建的-一个依赖关系图，可以对职程（Worker）的内部进行细粒度控制。

Celery集成常用的Web框架，详细如下：

| Web框架                                              | 集成包                                                     |
| ---------------------------------------------------- | ---------------------------------------------------------- |
| [Pyramid](https://trypyramid.com/documentation.html) | [pyramid-celery](https://pypi.org/project/pyramid-celery/) |
| [Pylons](https://pylonshq.com/)                      | [celery-pylons](https://pypi.org/project/celery-pylons/)   |
| [Flask](https://flask.palletsprojects.com/en/2.1.x/) | 不需要                                                     |
| [web2py](http://web2py.com/)                         | [web2py-celery](https://pypi.org/project/web2py-celery/)   |
| [Tornado](https://www.tornadoweb.org/en/stable/)     | [tornado-celery](https://pypi.org/project/tornado-celery/) |
| [Tryton](http://www.tryton.org/)                     | [celery_tryton](https://pypi.org/project/celery_tryton/)   |

?> 提示：集成包并不是必须安全的，但使用它们可以更加快速和方便的开发，有时它们会在fork(2) 中添加例如数据库关闭连接的回调。

### Celery安装

版本要求：使用Celery4.0版本以上，需要Python ❨2.7,3.4,3.5❩

使用pip进行安装：`pip install -U Celery`

Celery 自定义了一组用于安装 Celery 和特定功能的依赖，可以在中括号加入您需要依赖，并可以通过逗号分割需要安装的多个依赖包：

```
pip install "celery[librabbitmq]"
pip install "celery[librabbitmq,redis,auth,msgpack]"
```

序列化

- celery[auth]：使用auth保证程序的安全
- celery[msgpack]：使用msgpack序列化
- celery[yaml]：使用yaml序列化

并发

- celery[eventlet]：基于 [eventle](https://pypi.python.org/pypi/eventlet/) 的并发池
- celery[gevent]：基于 [gevent](https://pypi.python.org/pypi/gevent/) 的并发池

传输和后端

- celery[librabbitmq]：使用librabbitmq库
- celery[redis]：使用Redis进行消息传输或后端结果存储
- celery[sqs]：使用Amazon SQS进行消息传输（实验阶段）
- celery[tblib]：使用 task_remote_tracebacks 的功能
- celery[memcache]：使用Memcached作为后端结果存储（使用的是[pylibmc](https://pypi.python.org/pypi/pylibmc/)）
- celery[pymemcache]：使用Memcached作为后端结果存储（纯Python实现）
- celery[cassandra]：使用Apache Cassandra作为后端结果存储
- celery[couchbase]：使用CouchBase作为后端结果存储
- celery[arangodb]：使用ArangoDB作为后端结果存储
- celery[elasticsearch]：使用ElasticSearch作为后端结果存储
- celery[riak]：使用Riak作为后端结果存储
- celery[dynamodb]：使用AWS DynamoDB作为后端结果存储
- celery[zookeeper]：使用Zookeeper进行消息传输
- celery[sqlalchemy]：使用SQLlchemy作为后端结果存储（支持）
- celery[pyro]：使用Pyro4进行消息传输（实验阶段）
- celery[slmq]：使用 SoftLayer Message Queue进行消息传输（实验阶段）
- celery[consul]：使用Consul.io Key/Value进行存储传输消息或后端结果存储（实验阶段）
- celery[django]：支持比较低的Django版本，不建议您在项目中使用它，它仅供参考

## 上手使用

### Broker

Celery 需要消息中间件（Broker）来进行发送和接收消息。 

| 名称       | 状态     | 监控 | 远程控制 |
| ---------- | -------- | ---- | -------- |
| RabbitMQ   | 稳定     | 是   | 是       |
| Redis      | 稳定     | 是   | 是       |
| Amazon SQS | 稳定     | 否   | 否       |
| Zookeeper  | 实验阶段 | 否   | 否       |

**RabbitMQ 和 Redis 中间人的功能比较齐全，但也支持其它的实验性的解决方案，其中包括 SQLite 进行本地开发，但目前实验阶段的中间人（Broker）只是功能性的，但是没有专门的维护人员。**

缺少监控就意味着这个监控已经失效，因此相关的 Flower、Celery events、celerymon 和其他基于此功能的监控工具全部失效。

远程管理控制是指可以通过 celery inspect 和 celery control（以及使用远程控制API的工具）在程序运行时检查和管理职程（Worker）的能力。

#### RabbitMQ

**安装 RabbitMQ 服务**：可以通过 [RabbitMQ官网](https://www.rabbitmq.com/download.html) 进行 [安装RabbitMQ](https://www.rabbitmq.com/install.html) ，MacOS安装请查阅 [Mac OS安装RabbitMQ]()。

?> 提示：如果在安装 RabbitMQ 后，使用 rabbitmqctl 出现 nodedown 错误信息，可以查阅这片文章解决问题：http://www.somic.org/2009/02/19/on-rabbitmqctl-and-badrpcnodedown/

**创一个RabbitMQ账户**：

```shell
$ sudo rabbitmqctl add_user myuser mypassword
$ sudo rabbitmqctl add_vhost myvhost
$ sudo rabbitmqctl set_user_tags myuser mytag
$ sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
```

修改myuser、mypassword、myvhost为自己配置的配置信息。关于更多RabbitMQ配置，请查阅 [RabbitMQ手册](https://www.rabbitmq.com/admin-guide.html)。

**配置系统名称：如果您使用的 DHCP 随机分配的主机名称，需要重新永久化配置主机名称。因为 RabbitMQ 是使用主机名与各个节点进行通信的。**可以使用 **scutil** 命令进行永久配置主机名：

```shell
$ sudo scutil --set HostName myhost.local
```

然后将主机名添加到 /etc/hosts 中，以便进行解析：

```
127.0.0.1    localhost myhost myhost.local
```

如果您的 rabbitmq-server 已经在运行，您的节点名称现在应该是 rabbit@myhost ，可以通过 rabbitmqctl 进行验证查看：

```shell
$ sudo rabbitmqctl status
Status of node rabbit@myhost ...
[{running_applications,[{rabbit,"RabbitMQ","1.7.1"},
                    {mnesia,"MNESIA  CXC 138 12","4.4.12"},
                    {os_mon,"CPO  CXC 138 46","2.2.4"},
                    {sasl,"SASL  CXC 138 11","2.1.8"},
                    {stdlib,"ERTS  CXC 138 10","1.16.4"},
                    {kernel,"ERTS  CXC 138 10","2.13.4"}]},
{nodes,[rabbit@myhost]},
{running_nodes,[rabbit@myhost]}]
...done.
```

如果 DHCP 给您分配主机名称是以IP地址（如：23.10.112.31.comcast.net），RabbitMQ将尝试用 raabit@23 ：非法的用户名。

**启动RabbitMQ服务**：`$ sudo rabbitmqctl-server`

**后台启动RabbitMQ服务**：`$ sudo rabbitmqctl-server -detached`

**停止RabbitMQ服务**：永远不要通过 kill 命令来进行停止 RabbitMQ 运行，使用 `rabbitmqctl` 命令来进行停止 RabbitMQ，`$ sudo rabbitmqctl stop`

**配置Broker：RabbitMQ 是默认的中间人（Broker），只需要配置连接的URL即可，不需要安装额外的的配置以及初始化配置信息。**

```python
broker_url = 'amqp://myuser:mypassword@localhost:5672/myvhost'
```

有关 Celery 各种中间人（Broker）的配置列表，请查阅代理设置，并且按照说明设置用户名和密码。

#### Redis

使用 Redis 作为中间人（Broker）必须要安装 Celery 的依赖库，您可以通过 celery[redis] 进行安装：

```
pip install -U "celery[redis]"
```

Redis 的 URL 的格式为：URL 的所有配置都可以自定义配置的，默认使用的是 localhost 的 6379 端口中 0 数据库。（ Redis 默认有 16 个数据库）

```
redis://:password@hostname:port/db_number
```

Redis 的配置非常的简单，只需要配置 Redis 的 URL ：

```python
app.conf.broker_url = 'redis://localhost:6379/0'
```

可以通过 Uninx 套接字进行连接，URl 格式如下：

```
redis+socket:///path/to/redis.sock
```

可以通过设置 virtual_host参数添加到URL上进行指定使用时 Uninx 套接字连接的数据库编号：

```
redis+socket:///path/to/redis.sock?virtual_host=db_number
```

Celery 也可以连接 Redis 哨兵也是非常简单的：

```python
app.conf.broker_url = 'sentinel://localhost:26379;sentinel://localhost:26380;sentinel://localhost:26381'
app.conf.broker_transport_options = {'master_name':'cluster1'}
```

**可见性超时：可见性超时为将消息重新下发给另外一个程序之前等待确认的任务秒数。**可以通过 `broker_transport_options` 选项进行修改：默认的可见性超时时间为1个小时。

```python
app.conf.broker_transport_options = {'visibility_timeout': 3600} # 一个小时
```

**保存结果**：如果您想保存任务执行返回结果保存到Redis，您需要进行以下配置。

```python
app.conf.result_backend = 'redis://localhost:7379/0'
```

如果您使用的是 Redis 哨兵默认是，则需要使用 `result_backend_transport_options` 进行指定 master_name：

```python
app.conf.result_backend_transport_options = {'master_name': "mymaster"}
```

**广播前缀：默认情况下，所有的虚拟机都可以看到广播的消息。您必须为消息进行设置前缀，以便它们由仅活动的虚拟机接收。**

```python
app.conf.broker_transport_options = {'fanout_prefix': true}
```

!> 注意：该选项仅是向后兼容的，老版本不支持。集群中所有的worker都必须要开启设置，否则无法进行通信。

默认情况下， 职程（Worker）收到所有与任务相关的事件。为了避免该情况发生，需要进行配置 `fanout_patterns` 广播模式，以便职程（Worker）只能订阅相关的事件：

```python
app.conf.broker_transport_options = {'fanout_patterns': true}
```

**驱逐key**：在某些情况下，Redis会根据（驱逐策略）进行驱逐一些key，您可以在Redis服务器的 time_out 参数设置为0进行避免key被驱逐。

```
InconsistencyError: Probably the key ('_kombu.binding.celery') has been
removed from the Redis database.
```

#### 选择中间人

Celery 需要一个中间件来进行接收和发送消息，通常以独立的服务形式出现，成为 消息中间人（Broker）

**使用RabbitMQ**：[RabbitMQ](https://www.rabbitmq.com) 的功能比较齐全、稳定、便于安装。在生产环境来说是首选的，如果您使用的是 Ubuntu 或 Debian ，可以通过以下命令进行安装 RabbitMQ：

```
$ sudo apt-get install rabbitmq-server
```

如果在 Docker 中运行 RabbitMQ ，可以使用以下命令：

```
$ docker run -d -p 5462:5462 rabbitmq
```

命令执行完毕之后，中间人（Broker）会在后台继续运行，准备输出一条 *Starting rabbitmq-server: SUCCESS* 的消息。

如果您没有 Ubuntu 或 Debian ，你可以访问官方网站查看其他操作系统（如：Windows）的安装方式：http://www.rabbitmq.com/download.html

**使用Redis**：[Redis](https://redis.io) 功能比较全，但是如果突然停止运行或断电会造成数据丢失，有关 Celery 中使用 Redis 的详细信息：

```
$ docker run -d -p 6379:6379 redis
```

**其他中间人（Broker）**：除以上提到的中间人（Broker）之外，还有处于实验阶段的中间人（Broker），其中包含[ Amazon SQS]()。相关完整的中间人（Broker）列表，请查阅[`中间人：Broker`]()。

### 简单使用

**我们把创建 Celery 程序成为 Celery 应用或直接简称为 app。**

#### 简单demo

首先学习一个简单的demo，先来**创建名称 `tasks.py` 文件**其内容如下：

```python
from celery import Celery

# 第一个参数为当前模块的名称，只有在 `__main__` 模块中定义任务时才会生产名称。
# 第二个参数为中间人（Broker）的链接URL，RabbitMQ可以写为amqp://guest@localhost//（Celery默认使用的也是RabbitMQ），使用Redis可以写为 redis://localhost。
app = Celery('tasks', broker='redis://localhost')

# 创建了一个名称为add的任务，返回的俩个数字的和。
@app.task
def add(x, y):
    return x + y
```

#### 调度任务

现在可以使用 worker 参数进行执行我们刚刚创建执行单元（Worker）：

```shell
celery -A tasks worker --loglevel=info
```

?> 提示：在生产环境中，如果需要将职程（Worker）作为守护进程在后台运行，可以使用平台提供的工具来进行实现，或使用类似 supervisord 这样的工具来进行管理。

当然我们也可以通过 `delay()` （ `apply_async()` 的快捷方法）调用创建的实例任务，这样可以更好的控制任务的执行：

```python
from tasks import add

# 该任务已经有执行单元（Worker）开始处理，可以通过控制台输出的日志进行查看执行情况。
add.delay(4, 4)
```

调用任务会返回一个 AsyncResult 的实例，用于检测任务的状态，等待任务完成获取返回值（如果任务执行失败，会抛出异常）。默认这个功能是不开启的，如果开启则需要配置 Celery 的结果后端。

#### 保存结果

如果您需要跟踪任务的状态，Celery 需要在某处存储任务的状态信息。Celery 内置了一些后端结果：[SQLAlchemy/Django](https://www.sqlalchemy.org) ORM、[Memcached](http://memcached.org)、[Redis](https://redis.io)、 RPC ([RabbitMQ](https://www.rabbitmq.com)/AMQP)以及自定义的后端结果存储中间件。

针对本次实例，我们使用 RPC 作为结果后端，将状态信息作为临时消息回传。后端通过 backend 参数指定给 Celery（或者通过配置模块中的 result_backend 选项设定）：

```python
app = Celery('tasks', broker='pyamqp://', backend='rpc://')
```

现在已经配置结果后端，重新调用执行任务。会得到调用任务后返回的一个 AsyncResult 实例：

```python
result = add.delay(4, 4)
```

`ready()` 可以检测是否已经处理完毕：

```python
# 返回False代表未处理完，True就是处理完了
result.ready()
```

整个任务执行过程为异步的，如果一直等待任务完成，会将异步调用转换为同步调用：

```python
result.get(timeout=1)  # 8
```

如果任务出现异常，`get()` 会再次引发异常，可以通过 propagate 参数进行覆盖：

```python
result.get(propagate=False)
```

如果任务出现异常，可以通过以下命令进行回溯：

```python
result.traceback
```

!> 注意：如果后端使用资源进行存储结果，必须要针对调用任务后返回每一个 AsyncResult 实例调用 get() 或 forget() ，进行资源释放。

#### 配置

Celery 像家用电器一样，不需要任何配置，开箱即用。它有一个输入和输出，输入端必须连接中间人（Broker），输出端可以连接到结果后端。如果仔细观察一些家用电器，会发现有很多到按钮，这就是配置。

大多数情况下，使用默认的配置就可以满足，也可以按需配置。查看配置选项可以更加的熟悉 Celery 的配置信息，可以参考 [`配置和默认配置：Configuration and defaults`]() 章节阅读 Celery 的配置。

可以直接在程序中进行配置，也可以通过配置模块进行专门配置。例如，通过 task_serializer 选项可以指定序列化的方式：

```python
app.conf.task_serializer = 'json'
```

如果需要配置多个选项，可以通过 update 进行配置：

```python
app.conf.update(
    task_serializer='json',
    accept_content=['json'],  # Ignore other content
    result_serializer='json',
    timezone='Europe/Oslo',
    enable_utc=True,
)
```

针对大型的项目，建议使用专用配置模块，进行针对 Celery 配置。不建议使用硬编码，建议将所有的配置项集中化配置。集中化配置可以像系统管理员一样，当系统发生故障时可针对其进行微调。

可以通过 `app.config_from_object()` 进行加载配置模块：

```python
app.config_from_object('celeryconfig')
```

其中 celeryconfig 为配置模块的名称，这个是可以自定义修改的。

在上面的实例中，需要在同级目录下创建一个名为 `celeryconfig.py` 的文件，添加以下内容：

```python
broker_url = 'pyamqp://'
result_backend = 'rpc://'

task_serializer = 'json'
result_serializer = 'json'
accept_content = ['json']
timezone = 'Europe/Oslo'
enable_utc = True
```

可以通过以下命令来进行验证配置模块是否配置正确：

```
python -m celeryconfig
```

Celery 也可以设置任务执行错误时的专用队列中，这只是配置模块中一小部分，详细配置如下：

```python
task_routes = {
    'tasks.add': 'low-priority',
}
```

Celery 也可以针对任务进行限速，以下为每分钟内允许执行的10个任务的配置：

```python
task_annotations = {
    'tasks.add': {'rate_limit': '10/m'}
}
```

如果使用的是 RabbitMQ 或 Redis 的话，可以在运行时进行设置任务的速率：

```
$ celery -A tasks control rate_limit tasks.add 10/m
worker@example.com: OK
    new rate limit set successfully
```

有关远程控制以及监控职程（Worker），详情参阅 [`路由任务：Routing Tasks`]()了解更多的任务路由以及 task_annotations 有关的描述信息，或查阅 [`监控和管理手册：Monitoring and Management Guide`]()。





## 故障处理

“常见问题“中含有一部分故障排除信息。

**职程（Worker）无法正常启动：权限错误**

- 如果使用系统是 Debian、Ubuntu 或其他基于 Debian 的发行版：Debian 最近把 `/dev/shm/`重名 `/run/shm`。使用软连接可以解决该问题：

```python
# ln -s /run/shm /dev/shm
```

- 其他：如果设置了 `--pidfile` `--logfile` 或 `--statedb` 其中的一个参数，必须要保证职程（Worker）对指向的文件/目录可读可写。

**任务总处于 PENDING （待处理）状态**

所有任务的状态默认都是 PENDING （待处理）状态，Celery 在下发任务时不会更换任务状态， 并且如果没有历史任务的都是会被任务待处理状态。

1. 确认任务没有启用 ignore_result，如果启用，会强制跳过任务更新状态。
2. 确保 task_ignore_result 未启用。
3. 确保没有旧的职程（Worker）正在运行。启动多个职程（Worker）比较容易，在每次运行新的职程（Worker）之前需要确保之前的职程是否关闭。未配置结果后端的职程（Worker）是否正在运行，可能会消费当前的任务消息。`–pidfile` 参数设置为绝对路径，确保该情况不会出现。
4. 确认客户是否配置正确。可能由于某种场景，客户端与职程（Worker）的后端不配置不同，导致无法获取结果，所以需要确保配置是否正确：

```python
result = task.delay(…)
print(result.backend)
```



```
Celery和RabbitMQ是两个层面的东西。
Celery是一个分布式的任务队列。它的基本工作就是管理分配任务到不同的服务器，并且取得结果。至于说服务器之间是如何进行通信的？这个Celery本身不能解决。
所以，RabbitMQ作为一个消息队列管理工具被引入到和Celery集成，负责处理服务器之间的通信任务。

当然，后来Celery相继增加了一些对Redis,MongoDB之类的支持。原因是RabbitMQ尽管足够强大，但对于一些相对简单的业务环境来说可能太多（复杂）了一些。这样用户可以有多一些的选择。

回复
wingyiu：任务队列和消息队列区别？
