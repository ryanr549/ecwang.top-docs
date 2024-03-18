# 并行计算

一个简单(也可能愚蠢)的使用`multiprocessing`包进行并行计算的例子


```python
import numpy as np
import multiprocessing


threads = 40 # 设置并行进程数

# 定义一个函数，作为进程的执行任务（因此需要把要做的事情写成函数的形式）
def func1(a, b):
    result = np.sqrt(a ** 2 + b ** 2)
    print(f'Result is {result}.')
   
    
# （假设我们有一组参数列表）
a = []
b = []   
    
num = 5500 # 总循环次数

if __name__ == '__main__':
    for k in range(num // threads + 1): # 每组40个，每次运行1组
        all_process = []
        for i in range(threads):        # 添加40个进程到 list: ‘all_process’
            idx = threads * k + i
            if idx < num:
                all_process.append(multiprocessing.Process(target=func1, args=(a[idx], b[idx],)))
                # multiprocessing 的 Process 方法中，在 'target' 写需要运行的函数名，将参数输入 'args'

        for p in all_process:
            p.start()
        
        for p in all_process:
            p.join()
    
    print("# All processes have finished execution.")

```
