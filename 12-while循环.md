# while循环

### 关键字while

while表示在...期间

##### 格式

```
while 条件语句:
   循环体
```

while：表示一个while循环的声明

条件语句：结果是True，或者False

循环体：重复执行的代码段

执行过程：**只要条件语句结果为True就执行循环体，直到条件语句的值为False，就不再执行循环体，循环结束**。

**注意：当循环次数不确定的时候，使用while循环。**

**注意：若条件语句的结果一直都是True,就会造成死循环。所以在使用while循环时，循环体中要有结束循环的操作**

##### 使用方法

例子1：

```
while True:
	print('aaa')  

# 注释：在条件语句为True，循环体没有结束循环的操作，这就是一个死循环。
```

例子2：

```
flag = True
while flag:
    print('aaa')
    flag = False

输出：
aaa
# 注释：变量flag第一次赋值为True，while循环条件成立，执行下面的循环体，输出aaa，变量flag第二次赋值为False，while循环条件不成立，不执行循环体
```

### 关键字continue

continue表示继续下一次。

##### 使用方法

**只要运行continue，就结束当前这次循环，进入下一次循环**。（适用for循环、while循环）

```
for x in range(5):
    if x % 2:     # 筛选出奇数
        continue  # 结束这次循环，进入下次循环
    print(x)  	  # 0 2 4，打印偶数
```

### 关键字break

break表示跳出。

##### 使用方法

**只要运行break，就结束整个循环**。（适用for循环、while循环）

```
while True:
    print(0)     
    break		# 遇到break直接结束整个循环
print('结束')  	  

输出：
0
结束
```
