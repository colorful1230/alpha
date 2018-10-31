---
layout: article
title: ubuntu下安装tensorflow-gpu
tags: tensorflow
---

### 下载cuda

目前通过pip安装的tensorflow只支持cuda9.0，其它版本需要自己编译或者找别人编译好的。这里下载cuda9.0，[下载链接](https://developer.nvidia.com/cuda-90-download-archive)

<!--more-->

![cuda](https://upload-images.jianshu.io/upload_images/150061-cead1610626a724f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载好后运行
```bash
chmod 755 cuda_9.0.176_384.81_linux.run && sudo ./cuda_9.0.176_384.81_linux.run
```
安装过程中可以选择安装NVIDIA显卡驱动，如果直接在x service启动的情况下会报错，可以通过ctrl+alt+f2进到命令行界面进行安装，首先运行
```bash
sudo /etc/init.d/lightdm stop
```
然后就可以正常安装了，安装好后再运行
```bash
sudo /etc/init.d/lightdm start
```
就可以启动桌面环境。

### 安装tensorflow

tensorflow可以直接通过pip安装，推荐用python3，先安装pip3
```bash
sudo apt install python3-dev python3-pip
```
然后安装tensorflow
```
pip install --upgrade tensorflow-gpu
```

安装过程问题不大，有缺失的依赖安装一下就可以了。官方安装说明文档地址在[这里](https://tensorflow.google.cn/install/pip)

### 测试tensorflow

安装好之后可以运行以下代码进行测试

```bash
➜  ~ python3 -c "import tensorflow as tf; print(tf.__version__)"
python3: Relink `/lib/x86_64-linux-gnu/libudev.so.1' with `/lib/x86_64-linux-gnu/librt.so.1' for IFUNC symbol `clock_gettime'
[1]    15404 segmentation fault (core dumped)  python3 -c "import tensorflow as tf; print(tf.__version__)"
```
但是这里报了一个错
![timg.jpeg](https://upload-images.jianshu.io/upload_images/150061-fae6361a5d1a8f6b.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在官方文档和网上查了半天，原来是没有安装CNDNN，先下载CNDNN，[下载地址](https://developer.nvidia.com/rdp/cudnn-download](https://developer.nvidia.com/rdp/cudnn-download)

![cudnn](https://upload-images.jianshu.io/upload_images/150061-d53a4fb29fc3a238.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将下载好的文件拷贝到/usr/local/cuda-9.0目录下就可以了，再运行以下测试代码，之后就成功了。

```bash
➜  ~ python3 -c "import tensorflow as tf; print(tf.__version__)"
1.11.0
```

### 小结
以上就是在Ubuntu下安装tensorflow-gpu的简单方法，官方目前只支持cuda9.0，如果要使用更新的版本需要自己手动编译，比较复杂，后面有需要再试着编译，官方说明文档在[这里](https://tensorflow.google.cn/install/source)。