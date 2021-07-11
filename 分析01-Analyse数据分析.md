# Analyse数据分析

### 数据分析简介

##### 定义过程

基本定义：**基于商业目的对海量数据进行分析并提炼出有价值信息的一个过程。**

完整过程：**明确分析目的与框架、数据收集、数据处理（数据清洗、数据转换）、数据分析、数据展现和撰写报告等6个阶段。**

##### 语言优势

为什么用Python进行数据分析？

1. 拥有巨大活跃的科学计算社区；
2. 数据科学、机器学习、学界和工业界开发重要语言；
3. 胶水语言，轻松集成旧有算法和系统；
4. 不仅适用于研究和原型构建，同时也适用于构建生产系统。

##### 分析工具

Excel表格：最容易、最广泛的数据分析工具。

NumPy库：Python科学计算的基础包（全称：Numerical Python），使用快速高效的多维数组对象ndarray；在存储和处理数据时比内置的Python数据结构高效得多。用于对数组执行元素级计算以及直接对数组执行数学运算的函数；用于读写硬盘上基于数组的数据集的工具；线性代数运算、傅里叶变换，以及随机数生成；成熟的C API， 用于Python插件和原生C、C++、Fortran代码访问NumPy的数据结构和计算工具；

pandas

pandas提供了快速便捷处理结构化数据的⼤量数据结构和函数。

pandas兼具NumPy⾼性能的数组计算功能以及电⼦表格和关系型数据库（如SQL）灵活

的数据处理功能。它提供了复杂精细的索引功能，能更加便捷地完成重塑、切⽚和切块、

聚合以及选取数据⼦集等操作

数据操作、准备、清洗是数据分析最重要的技能（耗时最⻓）

3、matplotlib

最流⾏的⽤于绘制图表和其它⼆维数据可视化的Python库

适合创建出版物上⽤的图表

4、IPython 和 Jupyter

执⾏ 探索 ⼯作流 （探索、试错、重复）

IPython web notebook Jupyter notebook (⽀持40多种编程语⾔）

Jupyter notebook⽀持markdown和html

5、Scipy

⼀组专⻔解决科学计算中各种标准问题域的包的集合。

6、scikit-learn

scikit-learn成为了Python的通⽤机器学习⼯具包

7、statsmodels

statsmodels包含经典统计学和经济计量学的算法

常⽤模块引⽤惯例

import numpy as np

import matplotlib.pyplot as plt

import pandas as pd

import statsmodels as sm

安装Anaconda

Downloads - Anaconda

Python 1991年出现，Python 2.x在2020年就会到期（包括重要的安全补丁），新项⽬请使⽤

Python 3IPython基础

$ ipython

In [1]: a = 5

In [2]: a

Out[2]: 5

pretty printed（Python对象被格式化为更易读的形式）

In [5]: import numpy as np

In [6]: data = {i : np.random.randn() for i in range(7)}

In [7]: data

Out[7]: 

{0: -0.20470765948471295,

 1: 0.47894333805754824,

 2: -0.5194387150567381,

 3: -0.55573030434749,

 4: 1.9657805725027142,

 5: 1.3934058329729904,

6: 0.09290787674371767}

运⾏Jupyter Notebook

notebook是Jupyter项⽬的重要组件之⼀，它是⼀个代码、⽂本（有标记或⽆标记）、数据可

视化或其它输出的交互式⽂档。

Python的Jupyter内核是使⽤IPython。

启动Jupyter

$ jupyter notebookJupyter Notebook

要新建⼀个notebook，点击按钮New，选择“Python3”或“conda[默认项]”。如果是第⼀次，

点击空格，输⼊⼀⾏Python代码。然后按Shift-Enter执⾏。

当保存notebook时（File⽬录下的Save and Checkpoint），会创建⼀个后缀名为.ipynb的⽂

件。

要加载存在的notebook，把它放到启动notebook进程的相同⽬录内。

%pwd

%ls

Tab补全

In [3]: b = [1, 2, 3]

In [4]: b.<Tab>

b.append b.count b.insert b.reverse

b.clear b.extend b.pop b.sort

b.copy b.index b.remove

In [1]: import datetime

In [2]: datetime.<Tab>

datetime.date datetime.MAXYEAR datetime.timedelta

datetime.datetime datetime.MINYEAR datetime.timezone

datetime.datetime_CAPI datetime.time datetime.tzinfo

在变量前后使⽤问号？，可以显示对象的信息In [8]: b = [1, 2, 3]

In [9]: b?

Type: list

String Form:[1, 2, 3]

Length: 3

Docstring:

list() -> new empty list

list(iterable) -> new list initialized from iterable's items

In [10]: print?

Docstring:

print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

Prints the values to a stream, or to sys.stdout by default.

Optional keyword arguments:

file: a file-like object (stream); defaults to the current sys.stdout.

sep: string inserted between values, default a space.

end: string appended after the last value, default a newline.

flush: whether to forcibly flush the stream.

Type: builtin_function_or_method

def add_numbers(a, b):

 """

 Add two numbers together

 Returns

 \-------

 the_sum : type of arguments

 """

 return a + b

然后使⽤?符号，就可以显示如下的⽂档字符串：In [11]: add_numbers?

Signature: add_numbers(a, b)

Docstring:

Add two numbers together

Returns

\-------

the_sum : type of arguments

File: <ipython-input-9-6a548a216e27>

Type: function

使⽤??会显示函数的源码：

In [12]: add_numbers??

Signature: add_numbers(a, b)

Source:

def add_numbers(a, b):

 """

 Add two numbers together

 Returns

 \-------

 the_sum : type of arguments

 """

 return a + b

File: <ipython-input-9-6a548a216e27>

Type: function

搜索IPython的命名空间

In [13]: np.*load*?

np.__loader__

np.load

np.loads

np.loadtxt

np.pkgload%run, %load, %paste, %cpaste 命令

%run命令运⾏所有的Python程序

In [14]: %run test3.py

⽂件中所有定义的变量（import、函数和全局变量，除⾮抛出异常），都可以在IPython shell

中随后访问

在Jupyter notebook中，你也可以使⽤%load，它将脚本导⼊到⼀个代码格中

%load test3.py

%paste和%cpaste 函数。%paste 可以直接运⾏剪贴板中的代码

键盘快捷键魔术命令

%timeit 测量任何Python语句，例如矩阵乘法，的执⾏时间

In [23]: foo = %pwd

In [24]: foo

集成Matplotlib

IPython在分析计算领域能够流⾏的原因之⼀是它⾮常好的集成了数据可视化和其它⽤户界⾯

库，⽐如matplotlib

%matplotlib

import matplotlib.pyplot as plt

plt.plot(np.random.randn(50).cumsum())

数据类型：

表格型数据，其中各列可能是不同的类型（字符串、数值、⽇期等）。⽐如保存在关系型数据

库中或以制表符/逗号为分隔符的⽂本⽂件中的那些数据。

多维数组（矩阵）。

通过关键列（对于SQL⽤户⽽⾔，就是主键和外键）相互联系的多个表。

间隔平均或不平均的时间序列。