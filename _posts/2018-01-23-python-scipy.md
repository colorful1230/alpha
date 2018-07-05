---
layout: article
title: python数据处理(3)--scipy
tags: python
---

### scipy简介

scipy是在numpy的基础上增加了科学计算、工程计算等库函数，如线性代数、常微分方程和信号处理等函数。

<!--more-->

### 积分
scipy可以用于积分计算，首先引入需要的包

```
from scipy.integrate import quad, dblquad
```
其中quad用于一次积分，dblquad用于二次积分，除此之外还有nquad可以进行n次积分，下面主要介绍一次和二次积分

#### 一次积分
使用方法

```
quad(func, a, b)
```

其中func是func是要积分的函数，a和b是取值范围，这个方法的返回值是一个元组(value, delta)，value是积分值，delta是误差范围

```
(value, delta) = quad(lambda x: np.sin(x), 0, np.pi)
```
这段代码表示对f(x)=sin(x)进行积分，x的范围为[0, pi], 输出结果

```
2.0, 2.220446049250313e-14
```

#### 二次积分

```
dblquad(func, a, b, hfun, gfun)
```

其中func是func是要积分的函数，假设函数是f(x,y)，a和b是x的取值范围，hfun和gfun是y的取值范围，这个方法的返回值是一个元组(value, delta)，value是积分值，delta是误差范围

```
(value, delta) = dblquad(lambda x, y: np.sin(x) + np.cos(y), 0, np.pi, lambda y: 0, lambda y: np.pi /2)
```

这段代码表示对f(x, y) = sin(x) + cos(y), x的范围在[0, pi]，y的范围是[0, pi/2]，书称呼结果为

```
3.141592653589793, 4.214507361426152e-14
```

## 优化器

这里只进行简单的介绍，具体用法参考文档

### 最小值计算

```
minimize(func, x0)
```
其中func是目标函数，x0是取值的一个数组，首先定义一个复杂函数

```
def func(x):
    return sum(100 * (x[1:] - x[:-1] ** 2) ** 2 + (1 - x[:-1] ** 2))
```
然后定义一个数组用来计算最小值

```
x0=np.array([1.3, 0.7, 0.8, 1.9, 1.2])
```
最后计算最小值

```
minimize(func, x0, options={'disp': True})
```

```'disp': True```是打印出中间过程，结果如下

```
Current function value: -246023.210719
Iterations: 1000
Function evaluations: 8918
Gradient evaluations: 1274
[  2.17432105e+00   4.71801238e+00   2.22660709e+01   4.95732805e+02
   2.45749445e+05]
```

### 处理矩阵

scipy与numpy一样可以处理矩阵，下面通过代码举几个简单的例子进行介绍

```
# 定义一个2x2的矩阵
arr = np.array([[1,2], [2,3]])
print('行列式:', lg.det(arr))
print('逆矩阵:', lg.inv(arr))

b = np.array([3,5])
print('解方程组x+2y=3, 2x+3y=5', lg.solve(arr, b))

print('特征值', lg.eig(arr))

print('LU分解', lg.lu(arr))
print('QR分解', lg.qr(arr))
print('SVD分解', lg.svd(arr))
print('Schur分解', lg.schur(arr))
```

输出结果

```
行列式: -1.0
逆矩阵: [[-3.  2.]
 [ 2. -1.]]
解方程组x+2y=3, 2x+3y=5 [ 1.  1.]
特征值 (array([-0.23606798+0.j,  4.23606798+0.j]), array([[-0.85065081, -0.52573111],
       [ 0.52573111, -0.85065081]]))
LU分解 (array([[ 0.,  1.],
       [ 1.,  0.]]), array([[ 1. ,  0. ],
       [ 0.5,  1. ]]), array([[ 2. ,  3. ],
       [ 0. ,  0.5]]))
QR分解 (array([[-0.4472136 , -0.89442719],
       [-0.89442719,  0.4472136 ]]), array([[-2.23606798, -3.57770876],
       [ 0.        , -0.4472136 ]]))
SVD分解 (array([[-0.52573111, -0.85065081],
       [-0.85065081,  0.52573111]]), array([ 4.23606798,  0.23606798]), array([[-0.52573111, -0.85065081],
       [ 0.85065081, -0.52573111]]))
Schur分解 (array([[-0.23606798,  0.        ],
       [ 0.        ,  4.23606798]]), array([[-0.85065081, -0.52573111],
       [ 0.52573111, -0.85065081]]))

```
