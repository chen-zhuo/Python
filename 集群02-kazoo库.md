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

### 检查节点

Kazoo检查节点的最常用三个方法：

[`exists()`](https://kazoo.readthedocs.io/en/latest/api/client.html#kazoo.client.KazooClient.exists) checks to see if a node exists.

[`get()`](https://kazoo.readthedocs.io/en/latest/api/client.html#kazoo.client.KazooClient.get) fetches the data of the node along with detailed node information in a [`ZnodeStat`](https://kazoo.readthedocs.io/en/latest/api/protocol/states.html#kazoo.protocol.states.ZnodeStat) structure.

[`get_children()`](https://kazoo.readthedocs.io/en/latest/api/client.html#kazoo.client.KazooClient.get_children) gets a list of the children of a given node.