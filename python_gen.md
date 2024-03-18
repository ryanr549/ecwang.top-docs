# Python 列表生成式

在Python程序中经常要用到for循环，这有时会导致我们的程序显得很臃肿。

## 基础

例如我想生成一系列数的平方，用for循环是这样的：

```python
results = []
for i in range(5):
	results.append(i**2)
```

可以用列表生成式简化：

```python
results = [i**2 for i in range(5)]
```

这样就能把这个逻辑在一行里写完，也提升了可读性。总结就是

```python
[i的表达式 for i in 范围]
```

## 附加条件

此外还可以给表达式附加条件，从而筛选出我们想要的结果。例如求10以内偶数的平方：

```python
[i**2 for i in range(1, 11) if i % 2 == 0]
```

当然这里也可以通过改`range()`来达成目标。另举一例，我有一个fits文件，想要选出星表中红移`Z`大于0.5的源的`TARGETID`，for循环的写法是这样的：
```python
from astropy.io import fits
data = fits.getdata('catalog.fits')

id_list = []
for i in data:
	if i['Z'] > 0.5:
		id_list.append(i['TARGETID'])
```

也就是遍历`data`，用if语句判断符合条件的目标，再通过append添加到列表。类似于for循环中的if语句，我们可以在列表生成式中加上if：

```python
id_list = [i['TARGETID'] for i in data if i['Z'] > 0.5]
```

总结：

```python
[i的表达式 for i in 范围 if 想要挑选出的i满足的条件]
```

## 多层循环

还可以用于两层循环，例如我想展开一个二维列表：

```python
l_2d = [[1, 2], [3, 4, 5]]

l_1d = []
for i in l_2d:
	for j in i:
		l_1d.append(j)
```

用列表生成式：

```python
l_1d = [j for i in l_2d for j in i]
```
