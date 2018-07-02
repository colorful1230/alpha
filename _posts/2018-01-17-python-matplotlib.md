---
layout: article
title: python数据分析(2)--matplotlib
tags: python
---

##### 前情回顾
[python数据分析(1)--numpy](https://zhaolin1230.github.io/articles/2018/01/05/python-numpy.html)

### 简介
Matplotlib是一个Python的绘图库，可以绘制出常用图表，如散点图、柱状图等，下面做简要介绍。

<!--more-->

### 一个简单的demo

首先从一个demo看起，绘制三角函数并填充一定区域，每一步有注释说明，整体用法与MATLAB接近。

```python
# encoding=utf-8

import numpy
import matplotlib.pyplot as plot

def main():
    # 生成数组作为横坐标
    x = numpy.linspace(-numpy.pi, numpy.pi, 256, endpoint=True)
    cos = numpy.cos(x)
    sin = numpy.sin(x)
    plot.figure(1)

    # 绘制cos函数
    plot.plot(x, cos, color="blue", linewidth=1.0, linestyle='-', label='cos')
    # 绘制sin函数
    plot.plot(x, sin, 'r*', label='sin')
    # 设置标题
    plot.title('function')
    
    #------绘制坐标------
    ax = plot.gca()
    # 设置坐标四周的样式
    ax.spines['right'].set_color('none')
    ax.spines['top'].set_color('none')
    ax.spines['left'].set_position(('data', 0))
    ax.spines['bottom'].set_position(('data', 0))
    # 设置坐标值显示的位置
    ax.xaxis.set_ticks_position('bottom')
    ax.yaxis.set_ticks_position('left')

    # 设置坐标显示内容
    plot.xticks([-numpy.pi, -numpy.pi/2, 0, numpy.pi/2, numpy.pi],
                [r'$-\pi$', r'$-\pi/2$', r'$0$', r'$+\pi/2$', r'$+\pi$'])
    plot.yticks(numpy.linspace(-1, 1, 5, endpoint=True))

    # 设置每个坐标的样式
    for label in ax.get_xticklabels() + ax.get_yticklabels():
        label.set_fontsize(8)
        label.set_bbox(dict(facecolor='white', edgecolor='None', alpha=0.5))

    # 设置图例所在位置    
    plot.legend(loc='upper left')

    # 设置网格
    plot.grid()

    # 绘制填充区域
    plot.fill_between(x, numpy.abs(x) < 0.5, cos, cos > 0.5, color='blue', alpha=0.2)
    t = 1

    # 绘制直线
    plot.plot([t, t], [0, numpy.cos(t)], 'y', linewidth=3, linestyle='--')
    
    # 绘制注解
    plot.annotate('cos(1)', xy=(t, numpy.cos(1)), xycoords='data', xytext=(10, 30), textcoords='offset points',arrowprops=dict(arrowstyle='->',connectionstyle='arc3,rad=0.2'))
    
    plot.show()

if __name__ == '__main__':
    main()
```

主要解释一下fill_between()这个方法，这个方法完整定义为

```python
fill_between(x, y1, y2=0, where=None, interpolate=False, hold=None, **kwargs)
```
+ x-横坐标
+ y1-横坐标为x时填充纵坐标的起点
+ y2-横坐标为x时填充纵坐标的终点
+ where-纵坐标生效的填充范围

画出的图如下

![1.png](http://upload-images.jianshu.io/upload_images/150061-824bbcdd0a2bed86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 类型

matplotlib支持的绘图类型主要有以下几种

| 类型 | 说明 |
|---|---|
|scatter| 散点图 |
|bar | 柱状图|
|pie| 饼状图|
|polar| 极坐标|
|imshow| 热力图|

下面是每种图的示例。

#### 散点图

![2.png](http://upload-images.jianshu.io/upload_images/150061-4428c9c1a16307c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
def scatter():
    x = numpy.random.normal(0, 1, 100)
    y = numpy.random.normal(0, 1, 100)
    plot.scatter(x, y)
    plot.show()
```

#### 柱状图

![3.png](http://upload-images.jianshu.io/upload_images/150061-f6d1c2a15b630b10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```python
def bar():
    x = numpy.random.normal(0, 1, 100)
    y = numpy.random.normal(0, 1, 100)
    plot.bar(x, y, 0.2, edgecolor='white')
    plot.show()
```


#### 饼图

![4.png](http://upload-images.jianshu.io/upload_images/150061-b5c2171d550935d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
def pie():
    x = numpy.random.normal(0, 1, 5)
    y = numpy.random.normal(0, 1, 5)
    plot.pie(x)
    plot.show()
```

#### 极坐标

![5.png](http://upload-images.jianshu.io/upload_images/150061-8a97cc563e81cbb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
def polar():
    theta = numpy.arange(0, 2 * numpy.pi, 2 * numpy.pi / 20)
    r = 10 * numpy.random.rand(20)
    plot.polar(theta, r)
    plot.show()
```

#### 热力图

![](http://upload-images.jianshu.io/upload_images/150061-7041c5db4325e296.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```python
def heatmap():
    data = numpy.random.rand(3, 3)
    cmap = cm.Blues
    plot.imshow(data, interpolation='nearest', cmap=cmap, aspect='auto')
    plot.show()
```
### 小结
matplotlib的绘图功能十分强大和灵活，上面仅仅是很小的一部分，利用这个库在数据分析说结果之后就可以直接绘制出想要的图，不需要再用其它工具绘制，十分方便。
