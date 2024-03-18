# multiprocessing.Pool的应用

我们如何在自己的Python程序中使用并行（多进程）来达到加速的效果？正如[服务器概况](./server_guide)中所说的，我们的每个计算节点都有56个cpu核心可供使用；如何简单、有效地利用这些核，便是给程序加速的关键。本文结合我自己摸索好几天的经验，介绍用Python标准库multiprocessing实现多进程最方便的方法：进程池（Pool）。

# 多进程和多线程？

任务可以理解为*进程*（process），例如在自己的电脑上打开一个Word文档就是启动了一个Word进程。在一个Word进程中还需要执行打字输入、拼写检查、打印等子任务，我们把进程中的这些子任务称为*线程*（thread）。

一个进程中至少有一个线程，复杂进程中可以包含多个线程。这样一来，想要让我们的程序同步执行多个任务*应该有多线程和多进程两种思路*。一般来说，
- 对CPU密集型代码（例如循环计算），多进程效率更高，这是由于能够充分利用多个CPU核的性能——这恰恰符合我们科学计算的需求；
- 对IO密集型代码（比如硬盘频繁读写、网络爬虫），多线程效率更高。

更细的暂时不写了，我们直接进入实践部分。

# 多进程的实现

这是一个模拟科学计算的例子：

```python
import numpy as np
import time

def square_x(x):
    print(f'{x} squared is {x**2}')
    time.sleep(1)
    return x**2

results = []
start = time.time()
for i in range(4):
    results.append(square_x(i))

print(results)
print(f'calculation costs {time.time()-start} seconds.')
```

这里定义了函数`square_x`，为了模拟较大的运算负载我们用`time.sleep(1)`来让运行这个函数需要花费至少一秒。在后面的`for`循环中，我们模拟串行（serial）执行`square_x`函数，并返回运行花费的总时长。输出应当如下：

```text
1 squared is 1
2 squared is 4
3 squared is 9
[0, 1, 4, 9]
calculation costs 4.000613689422607 seconds.
```

要是能让四个CPU核分别负责执行这几个函数，应当能让运行时间减到1秒左右。

## 创建进程池

>`Pool`类可以提供指定数量的进程供用户调用，当有新的请求提交到Pool中时，如果池还没有满，就会创建一个新的进程来执行请求。如果池满，请求就会告知先等待，直到池中有进程结束，才会创建新的进程来执行这些请求。

可以理解进程池`Pool`是一个进程管家，你只要把任务给它，它就会想办法帮你尽量高效地执行。

```python
from multiprocessing import Pool

calculation = Pool(processes=4)
```

在上面的代码中，创建了一个名为`calculation`的进程池，它最多可以容纳4个进程。如果不填，它会自动获取可用的CPU核心数并创建一个最多可容纳核数个进程的进程池。

接下来介绍几种`Pool`类的方法以实现提交任务并行执行：
- `Pool.map(func, iterable, chunksize=None)`
- `Pool.starmap(func, iterable, chunksize=None)`
- `Pool.apply_async(func, args=(), callback=None`
- ~~`Pool.apply`~~
- ~~`Pool.map_async`~~
- `Pool.close()`：关闭进程池（停止接受任务）
- `Pool.join()`：等所有子任务完成后再继续执行本代码后面的主任务代码

# `Pool.map()`

这个方法是提交任务最基本的方式，首先给出用这种方法给前面例子实现并行的代码：

```python
from multiprocessing import Pool
import time

def square_x(x):
    print(f'{x} squared is {x**2}')
    time.sleep(1)
    return x**2

calculation = Pool(processes=4)
results = []
start = time.time()
results = calculation.map(square_x, range(4))
calculation.close()
calculation.join()
print(results)
print(f'calculation costs {time.time()-start} seconds.')
```

运行代码有这样的输出，可见的确实现了并行。

```text
0 squared is 0
1 squared is 1
3 squared is 9
2 squared is 4
[0, 1, 4, 9]
calculation costs 1.0058555603027344 seconds.

```

## 参数和返回值的解释

```python
results = calculation.map(square_x, range(4))
```

- `map()`接受的第一个参数：一个只接收一个实参的函数`func`，可以有返回值
- `map()`接受的第二个参数：一个可迭代对象，例如一个列表，`range(4)`都是可迭代对象。

>一个简单的判别方法是在
>```python
>for i in \<xxx>:
>    print(i)
>```
>语句中，能放在`<xxx>`位置的都是可迭代对象。

- `results = calculation.map(square_x, range(4))`这一句中，`Pool.map()`返回值是函数`func`对可迭代对象的每一项处理得到返回值的列表。从输出结果可以看出，虽然计算是并行执行，最终输出的结果`results`仍然保持了输入可迭代对象的顺序，对这一点可以放心。

## 函数需要接受多个参数

这里出现一个问题：如果我的函数需要接受多个实参怎么办？例如

```python
def add(x, y):
	print(f'{x} + {y} = {x + y}')
	return x + y

a = range(4)
b = [666, 233, 7, 8]
```

我们可以将要输入的两个可迭代对象两两一组打包成一个新的可迭代对象，具体做法是

```python
zip_args = list(zip(a, b))
```

然后再将`Pool.map()`换成`Pool.starmap()`就可以了，完整代码如下

```python
from multiprocessing import Pool
import time

def add(x, y):
    print(f'{x} + {y} = {x + y}')
    time.sleep(1)
    return x + y

a = range(4)
b = [666, 233, 7, 8]
zip_args = list(zip(a, b))
calculation = Pool(processes=4)
results = []
start = time.time()
results = calculation.starmap(add, zip_args)
calculation.close()
calculation.join()
print(results)
print(f'calculation costs {time.time()-start} seconds.')
```

同样成功实现了并行

```text
0 + 666 = 666
1 + 233 = 234
2 + 7 = 9
3 + 8 = 11
[666, 234, 9, 11]
calculation costs 1.0063591003417969 seconds.
```

注意，只是打包了参数可迭代对象而还是用`Pool.map()`是不行的，会报错。

# `Pool.apply_async()`

`Pool.apply`并不能实现并行，于是直接介绍它的非阻塞版本。先上代码：注意我将函数中sleep的时间变为随机，让我们看看它对输出的影响

```python
from multiprocessing import Pool
import time
import random

def square_x(x):
    print(f'{x} squared is {x**2}')
    time.sleep(random.random())
    return x**2

calculation = Pool(processes=4)
results = []
start = time.time()
for i in range(4):
    calculation.apply_async(square_x, (i,), callback=results.append)
calculation.close()
calculation.join()
print(results)
print(f'calculation costs {time.time()-start} seconds.')
```

输出如下：

```text
0 squared is 0
1 squared is 1
2 squared is 4
3 squared is 9
[9, 1, 0, 4]
calculation costs 0.8757891654968262 seconds.
```

可以看出，由于每次运行的时长不同（这与现实任务是相符合的），`results.append`的顺序与输入并不一致。所以只有在你不在意输出次序的时候用这种方法才合理。
## 参数和返回值的解释
```python
calculation.apply_async(square_x, args=(i,), callback=results.append)
```

- 第一个参数：函数，可以接受多个实参
- 第二个参数：必须是一个列表/元组，也就是说哪怕你函数只接受一个实参也得用（圆或方）括号***括起来***并***加逗号***
- 第三个参数：`callback=func()`，可以理解是一个只接受一个实参的函数

>本例中我就想对每个返回值`r`都使用
>```python
>results.append(r)
>```
>就可以写成`callback=results.append`

# `Pool.close()` & `Pool.join()`

这两句在使用`Pool()`时是必须的，否则可能会遇到意想不到的报错。还有另一种解决方法就是用`with`语句：

```python
with Pool(4) as p:
	p.map(...)
```
这样以来Pool只在with后面缩进的范围里生效，到了外面自然就关闭了...

有这几个方法基本就够用了。


