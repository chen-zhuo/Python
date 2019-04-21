# 匿名函数lambda

##### 定义

匿名函数：本质还是函数，以另外一种简单的方式来声明的

##### 关键字lambda

lambda：声明匿名函数

##### 匿名函数格式

函数名 = lambda 参数列表：返回值（**不需要return**）

##### 使用方法

普通函数

```
def my_sum1(x,y):
    return x+y

print(my_sum1(10, 20))			# 30
```

匿名函数

```
my_sum2 = lambda x, y: x+y

print(my_sum2(10, 20))			# 30
```
