---
layout: article
title: python数据分析(1)--numpy
tags: python
---


### 简介
NumPy系统是Python的一种开源的数值计算扩展，可用来存储和处理大型矩阵。

<!--more-->

### numpy中的array

ndarray是numpy提供的一种数组，python的数组可以存放各种类型和对象，运行速度下降，而ndarray为了提高运行速度，底层用c实现，限定只能使用一种类型。ndarray有以下几个常用属性

| 属性 | 说明 |
| ---|---|
|shape| 几行几列 |
|ndim| 数组维度 |
|dtype | 数组元素的类型 |
| itemsize| 数组中每个元素的大小 |
| size| 数组中元素的个数 |

下面分别打印以下这几个属性

```python
list = [[1,2,3], [4,5,6], [7,8,9]]
# 新建一个元素类型为int64的数组
numpy_list = numpy.array(list, dtype=numpy.int64)
print(numpy_list.shape)
print(numpy_list.ndim)
print(numpy_list.dtype)
print(numpy_list.itemsize)
print(numpy_list.size)
```

结果如下

```
(3, 3)  
2  
int64
8
9
```

### 常用array

| array | 说明 |
| ---|---|
| numpy.zeros[row, col] | 全零数组 |
|numpy.ones[row, col] | 全一数组 |
|numpy.random.rand| 生成随机数 |
| numpy.random.randInt(low, hight)| 生成指定范围内的随机整数|
|numpy.random.randn| 正态分布随机数 |
|numpy.random.choice()| 从指定的数组中生成随机数 |
|numpy.random.beta| beta分布|

```python
print(numpy.zeros([2, 3]))
print(numpy.ones([3, 5]))
print(numpy.random.rand())
print(numpy.random.randint(1,2))
print(numpy.random.randn())
print(numpy.random.beta(1, 1, 2))
```

```
[[ 0.  0.  0.]
 [ 0.  0.  0.]]
[[ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]]
0.6256807713479122
1
1.1866001408917326
[ 0.68579746  0.93474434]
```

### 常用操作

以下的list都是numpy的array

| 函数 | 说明 |
|---|---|
|numpy.arange(low, high) | 产生等差数列，范围[1,10)|
| list.reshape([row, col])| 将数组转换成指定行列的数组 |
| numpy.exp(list) | 指数 |
| numpy.sin(list) | 三角函数|
| numpy.log(list) | 对数|
| list.sum(axis)| 求和，axis指对数组深入的程度|
|list.max(axis)| 最大值 |
|list.min(axis)| 最小值|
|list1+list2| 数组元素相加|
|list1-list2| |
|list1*list2| |
|list1/list2| |
|numpy.dot() | 矩阵点乘 |
|numpy.concatenate(list1, list2) | 追加数组|
|numpy.vstack(list1, list2)|  |
|numpy.hstack(list1, list2)|  |
|munpy.split(list, n)| 将数组切分成n份 |
|numpy.copy(list)| 拷贝array|

测试程序

```
list = numpy.arange(1, 9)
print("等差数列")
print(list)
#-1为默认值
print("转换多维数组")
print(list.reshape(2,-1))
print("指数计算")
print(numpy.exp(list.reshape(2, -1)))
l = [[1,2,3], [4,5,6], [7,8,9]]
numpy_list = numpy.array(l)
print("求和计算")
# 深度为1的结果
print("axis=0")
print(numpy_list.sum(axis=0))
# 深度为2的结果
print("axis=1")
print(numpy_list.sum(axis=1))

print("数组相加")
list1 = numpy.array([[1,2,3], [4,5,6]])
list2 = numpy.array([[2,3,4], [3,4,5]])
print(list1 + list2)
```

输出结果

```
等差数列
[1 2 3 4 5 6 7 8]
转换多维数组
[[1 2 3 4]
 [5 6 7 8]]
指数计算
[[  2.71828183e+00   7.38905610e+00   2.00855369e+01   5.45981500e+01]
 [  1.48413159e+02   4.03428793e+02   1.09663316e+03   2.98095799e+03]]
求和计算
axis=0
[12 15 18]
axis=1
[ 6 15 24]
数组相加
[[ 3  5  7]
 [ 7  9 11]]
```

### 矩阵操作和线性方程组

| 函数 | 说明 |
|---|---|
|numpy.eye()| 生成单位矩阵 |
|inv(list)| 求逆矩阵|
|list.transpose()| 求转置矩阵|
|det(list)| 求矩阵的行列式 |
|eig(list)| 求特征值和特征向量，返回值第一个元组为特征值，第二个元组是特征向量|
|solve(list, y)| 求解方程组|

测试程序

```python
#encoding-utf8

import numpy
from numpy.linalg import *

def main():
    print("单位矩阵")
    print(numpy.eye(3))
    
    list = numpy.array([[3, 5], [7, 9]])
    print("逆矩阵")
    print(inv(list))
    print("转置矩阵")
    print(list.transpose())
    print("矩阵行列式")
    print(det(list))
    print("特征值和特征向量")
    print(eig(list))

    print("求解二元方程: 3x+7y=2, 5x+9y=5")
    y = numpy.array([[2], [5]])
    print(solve(list, y))
    pass
    
if __name__ == '__main__':
    main()
```

输出结果

```
单位矩阵
[[ 1.  0.  0.]
 [ 0.  1.  0.]
 [ 0.  0.  1.]]
逆矩阵
[[-1.125  0.625]
 [ 0.875 -0.375]]
转置矩阵
[[3 7]
 [5 9]]
矩阵行列式
-8.0
特征值和特征向量
(array([ -0.63324958,  12.63324958]), array([[-0.80897568, -0.46067886],
       [ 0.58784211, -0.88756689]]))
求解二元方程
[[ 0.875]
 [-0.125]]
```

### 其它

| 函数| 说明|
|---|---|
|numpy.fft.fft()| FTT计算|
|numpy.corrcoef()| 计算相关系数 |
|numpy.poly1d()| 生成一元多次函数|

测试程序

```python
#encoding-utf8

import numpy

def main():
    print("FFT")
    print(numpy.fft.fft(numpy.ones([1,6])))
    print("相关系数")
    print(numpy.corrcoef([1,0,1], [0,1,0]))
    print("生成一元多次函数")
    print(numpy.poly1d([1,2,3]))
    pass
    
if __name__ == '__main__':
    main()
```

输出结果

```
FFT
[[ 6.+0.j  0.+0.j  0.+0.j  0.+0.j  0.+0.j  0.+0.j]]
相关系数
[[ 1. -1.]
 [-1.  1.]]
生成一元多次函数
   2
1 x + 2 x + 3
```

### 小结
以上就是numpy简单的使用，numpy主要用于对矩阵的操作和变换，对矩阵计算简化过程，可以提供很大的便利。