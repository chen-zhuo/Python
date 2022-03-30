# Celery任务队列

## 简介

### 任务队列

**任务队列一般用于线程或计算机之间分配工作的一种机制。**任务队列的输入是一个称为任务的工作单元，有专门的工人（Worker）进行不断的监视任务队列，进行执行新的任务工作。

### Celery介绍

**Celery 是一款 Python 编写的非常简单、灵活、可靠的分布式消息队列工具，可用于处理大量消息，处理实时数据以及任务调度，并且提供了一整套操作此系统的一系列工具。**Celery可以在一台机器上运行，也可以在多台机器上运行，甚至可以跨数据中心运行，用来提高Celery的高可用性以及横向扩展能力。

一个完整的Celery工作流程当中，有四个部分：**消息生产者（producer）、消息中间件（message broker）、 任务执行单元（worker）、 任务结果存储（result store）。**

消息生产者（producer）：**提供消息的客户端。**

消息中间件（broker）：**Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。**包括，RabbitMQ，Redis，MongoDB，SQLAlchemy等，其中RabbitMQ、Redis比较稳定，其他处于测试阶段。

任务执行单元（worker）：**worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。**

任务结果存储（result store）：**存储worker执行的任务的结果，支持AMQP、Redis、MongoDB、MySQL、SQLAlchemy、Django ORM、Apache Cassandra、Elasticsearch等主流数据库。**

工作流程：**启动一个任务，客户端向消息队列发送一条消息，然后中间人（Broker）将消息传递给某一个工人（Worker），工人（Worker）执行中间人（Broker）分配的任务，最后将执行结果进行存储（Storage）。**

![QQ截图20220329142946](image/QQ截图20220329142946.png)

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

