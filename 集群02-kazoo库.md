# kazoo库

**Kazoo使用纯Python实现Zookeeper协议，因此不需要安装任何额外的软件就可以直接连接Zookeeper。**

## 简单使用

### 安装kazoo库

安装Kazoo也很简单，直接通过 `pip` 安装：

```
$ pip install kazoo
```

### 创建连接

要开始使用 Kazoo，必须创建一个 `KazooClient` 对象并建立连接：

```python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
```

?> 默认情况下，客户端将通过默认端口(2181)连接到本地Zookeeper服务器。首先您应该确保Zookeeper在那里运行，否则start命令将一直等到其默认超时。

连接后，无论间歇性连接丢失或 Zookeeper 会话过期，客户端都将尝试保持连接。可以通过调用stop指示客户端断开连接：

```python
zk.stop()
```

### 监听状态

**客户端的常见状态可以分为三种：丢失（LOST）、连接（CONNECTED）、暂停（SUSPENDED）。**可以通过查看 `state` 属性来确定：

```python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
print(zk.state)  # CONNECTED 注释：当前处于连接状态
# 注释：KazooClient首次创建实例，它是在LOST状态。建立连接后，它转换到 CONNECTED状态。
```

当客户端状态为SUSPENDED时，如果客户端正在执行需要与其他系统达成一致的操作（例如使用Lock），它就会暂停它正在执行的操作。重新建立连接后，客户端状态为CONNECTED时，客户端可以继续执行操作。

当客户端状态为LOST时，Zookeeper 将删除任何已创建的临时节点，这将会影响所有创建临时节点（例如使用Lock），当状态再次转换为CONNECTED后，需要重新获取锁。

常见的状态转换：

- **LOST -> CONNECTED**：新连接，或之前丢失的连接正在连接。
- **CONNECTED -> SUSPENDED**：与服务器的连接丢失发生在连接上。
- **CONNECTED -> LOST**：仅当在建立连接后提供无效的身份验证凭据时才会发生。
- **SUSPENDED -> LOST**：连接恢复到服务器，但由于会话过期而丢失。
- **SUSPENDED -> CONNECTED**：丢失的连接已恢复。

针对客户端状态，通过 `KazooState` 来进行判断可以写出如下状态监听器：

```python
from kazoo.client import KazooState

def my_listener(state):
    if state == KazooState.LOST:
        # Register somewhere that the session was lost
    elif state == KazooState.SUSPENDED:
        # Handle being disconnected from Zookeeper
    else:
        # Handle being connected/reconnected to Zookeeper

zk.add_listener(my_listener)
```

?> 在使用 `kazoo.recipe.lock.Lock` 或创建临时节点时，强烈建议添加状态侦听器，以便您的程序可以正确处理连接中断或 Zookeeper 会话丢失。

## 节点CRUD

Zookeeper 有用于创建、读取、更新和删除 Zookeeper 节点（这里称为 znodes 或节点）的功能。针对的，Kazoo 添加了几个方便的方法和一个更 Pythonic 的 API。

**首先需要明白的一点就是，节点是一个抽象的事物存在于会话当中，因此节点的路径也不是真实存在的。**

### 创建节点

Kazoo创建节点的最常用两个方法：

`ensure_path()`： 递归创建节点和沿途所需路径中的任何节点，但不能为节点设置数据，只能设置 ACL。

`create()`： 创建一个节点，并可以在节点上设置数据以及一个 watch 函数。它要求它的路径首先存在，除非 `makepath` 选项设置为True。

```python
# 创建一个节点，前提是该路径存在
zk.ensure_path("/my/favorite")

# 创建一个带有数据的节点
zk.create("/my/favorite/node", b"a value")
```

?> 子节点可以被无限递归的创建。

### 检查节点

Kazoo检查节点的最常用三个方法：

`exists()`：检查节点是否存在。

`get()`：获取节点的数据以及 `ZnodeStat` 结构中的详细节点信息。

`get_children()`：获取给定节点的子节点列表。

```python
# 判断节点
if zk.exists("/my/favorite"):
    # Do something

# 打印一个节点的版本和它的数据
data, stat = zk.get("/my/favorite")
print("Version: %s, data: %s" % (stat.version, data.decode("utf-8")))

# 列出子节点
children = zk.get_children("/my/favorite")
print("There are %s children with names %s" % (len(children), children))
```

### 更新节点

`set()`：更新指定节点的数据，在更新数据之前需要匹配提供的节点版本，否则将引发`BadVersionError` 而不是更新。

```python
zk.set("/my/favorite", b"some data")
```

### 删除节点

`delete()`：删除一个节点，也可以选择递归删除该节点的所有子节点。在删除节点之前需要匹配节点的版本，否则将引发 `BadVersionError` 而不是删除。

```python
zk.delete("/my/favorite/node", recursive=True)
```

## 重试连接

### retry方法

如果 Zookeeper 服务器出现故障或无法访问，与 Zookeeper 的连接可能会中断。默认情况下，kazoo 不会重试命令，因此这些失败将导致引发异常。为了帮助解决失败，kazoo 附带了一个 `retry()` 助手，如果 Zookeeper 连接异常被触发，重试连接到Zookeeper。

```python
result = zk.retry(zk.get, "/path/to/node")
```

### 重试函数

**某些命令可能具有独特的行为，不保证在每个命令的基础上自动重试。**例如，当使用临时和序列选项集创建一个节点，在命令成功返回之前可能会丢失连接，但该节点实际上已创建，再次运行时引发`kazoo.exceptions.NodeExistsError`。

由于 `retry()` 方法需要调用一个函数及其参数，因此可以将运行多个 Zookeeper 命令的函数传递给它，以便在连接丢失时重试整个函数。锁实现中的这个片段显示了它如何使用重试重新运行获取锁的函数，并检查它是否已经被创建来处理这种情况：

```python
# kazoo.recipe.lock snippet

def acquire(self):
    """获取互斥锁，阻塞直到它被获取""" 
    try:
        self.client.retry(self._inner_acquire)
        self.is_acquired = True
    except KazooException:
        # if we did ultimately fail, attempt to clean up
        self._best_effort_cleanup()
        self.cancelled = False
        raise

def _inner_acquire(self):
    self.wake_event.clear()

    # 确保我们指定的节点存在
    if not self.assured_path:
        self.client.ensure_path(self.path)

    node = None
    if self.create_tried:
        node = self._find_node()
    else:
        self.create_tried = True

    if not node:
        node = self.client.create(self.create_path, self.data,
            ephemeral=True, sequence=True)
        # 去除节点的路径
        node = node[len(self.path) + 1:]
```

### 自定义重试

有时，您可能希望为与 `retry()` 方法不同的命令或命令集设置特定的重试策略 。您可以使用您喜欢的特定重试策略手动创建 `KazooRetry` 实例：

```python
from kazoo.retry import KazooRetry

kr = KazooRetry(max_tries=3, ignore_expire=False)
result = kr(client.get, "/some/path")
```

这将重试`client.get`命令最多 3 次，并在发生时引发会话过期。您还可以使用默认行为创建一个实例，在重试期间忽略会话过期。

## 观察者

**Kazoo 可以在节点上设置监视功能，该功能可以在更改或删除节点或其子节点更改时触发。**

Watchers 可以通过两种不同的方式设置：

**第一种是 Zookeeper 默认支持一次性 watch 事件样式。**这些 watch 函数会被 kazoo 调用一次，并且不接收 session 事件。使用此样式需要将 watch 函数传递给`get()`、`get_children()`、`exists()`方法之一，当节点上的数据发生变化或节点本身被删除时，将调用传递给该方法的 watch 函数。它将传递一个 `WatchedEvent`实例：

```python
def my_func(event):
    # 检查子节点现在情况
    pass

# 当子节点改变调用watch事件对应的my_func函数
children = zk.get_children("/my/favorite/node", watch=my_func)
```

**第二种是使用 Kazoo 包含的一个用于监视数据和子节点修改的 API。**在每次触发事件时它不需要重新设置监视，即使用此 API 注册的 Watch 函数将在每次发生更改时立即调用，或者直到函数返回 False。如果 `allow_session_lost` 设置为 `True`，则在会话丢失时将不再调用该函数。以下方法 `ChildrenWatch`、`DataWatch` 可直接在 `KazooClient` 实例上使用，并且在以这种方式使用时不需要传入客户端对象，通过实例可以直接调用，允许它们用作装饰器：

```python
@zk.ChildrenWatch("/my/favorite/node")
def watch_children(children):
    print("Children are now: %s" % children)
# Above function called immediately, and from then on

@zk.DataWatch("/my/favorite")
def watch_node(data, stat):
    print("Version: %s, data: %s" % (stat.version, data.decode("utf-8")))
```

## 命令流

**Zookeeper 3.4 及更高版本支持一次发送多个命令，这些命令将作为单个原子单元提交。他们要么全部成功，要么全部失败。事务的结果将是事务中每个命令的成功/失败结果列表。**

```python
# transaction()方法返回一个TransactionRequest实例
transaction = zk.transaction()
# 调用它的方法来排队完成事务中的命令
transaction.check('/node/a', version=3)
transaction.create('/node/b', b"a value")
# 当事务准备好发送时，调用它的commit()方法。
results = transaction.commit()
```

在上面的示例中，只要有一个命令不可用，就会失败报错。这可以用来检查特定版本的节点，节点 `/node/a` 版本必须是3，如果节点与它应该处于的版本不匹配，将不会创建 `/node/b` ，则该版本事务失败。

## 异步使用

所有异步 Kazoo API 依赖于异步方法返回的 `IAsyncResult` 对象。可以使用 `rawlink()` 方法添加回调，无论使用线程还是使用 gevent 等异步框架，该方法都可以兼容。

连接处理创建连接：

```python
from kazoo.client import KazooClient
from kazoo.handlers.gevent import SequentialGeventHandler

# 使用IHandler接口来抽象回调系统
zk = KazooClient(handler=SequentialGeventHandler())

# 立即返回
event = zk.start_async()

# 设置超时等待30秒
event.wait(timeout=30)

if not zk.connected:
    # 未连接，停止尝试连接
    zk.stop()
    raise Exception("Unable to connect.")
```

?> Kazoo 不依赖 gevents/eventlet 猴子补丁，并要求传入适当的处理程序，默认处理程序是 `SequentialThreadingHandler`。

除了 `start_async()` 之外的所有kazoo _async方法都返回一个 `IAsyncResult`实例。回调函数将传递 `IAsyncResult` 实例，并应调用其上的 `get()`方法以检索值。如果异步函数遇到错误，则此调用可能会导致引发异常。它应该被抓住并妥善处理。

例子：

```python
import sys
from kazoo.exceptions import ConnectionLossException
from kazoo.exceptions import NoAuthException

def my_callback(async_obj):
    try:
        children = async_obj.get()
        do_something(children)
    except (ConnectionLossException, NoAuthException):
        sys.exit(1)

# 这两个语句立即返回，第二个设置一个回调
# 当 get_children_async 有它的返回值
async_obj = zk.get_children_async("/some/node")
async_obj.rawlink(my_callback)
```

除了返回 `IAsyncResult`对象之外，以下 CRUD 方法的工作方式都与它们的同步对应方法相同。

- 创建方法：`create_async()`
- 检查方法：`exists_async()`、`get_async()`、`get_children_async()`
- 更新方法：`set_async()`
- 删除方法：`delete_async()`

!> 注意 `ensure_path()` 具有目前没有异步版本也不能使用 `delete_async()`方法来执行递归删除。