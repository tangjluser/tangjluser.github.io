---
layout: blog_post
title: "PyTorch 学习笔记"
date: 2026-07-26
tags:
  - PyTorch
  - 深度学习
  - 学习笔记
---
# dataset 和 dataloader 的区别
- dataset 就是带标签 带序号的 数据集（数据表）
- dataloader 可以看作是对dataset的一个封装类  它提供两个功能，一个是获取指定的数据，另外一个是获取总共有多少数据 对应  getitem  和 getlen
![](/assets/img/blog/pytorch-learning/pytorch-note-01.png)
![](/assets/img/blog/pytorch-learning/pytorch-note-02.png)
dataset就是一个数据集，dataloader就像一个数据装载区，它从数据集里取点数据然后加到神经网络里去
# tensorboard的使用
- writer.add_scalar()的使用以及查看
- writer.add_image()的使用以及查看
# transforms的使用
- transforms具体是什么？它就是一个工具
![](/assets/img/blog/pytorch-learning/pytorch-note-03.png)
- ToTensor的使用
![](/assets/img/blog/pytorch-learning/pytorch-note-04.png)为什么要这么繁琐 先 tool = transforms.ToTensor()  然后在像调用函数一样去调用呢？
一方面是因为它在类中重写了  __ call __  方法，所以就可以这样
另外就是transforms它是一个工具这个思想，这样写也符合规范 我先拿个工具出来 再去使用它
同时还有以下好处：
![](/assets/img/blog/pytorch-learning/pytorch-note-05.png)
# transforms.Compose()的用法
- 它相当于是将一些列transforms的操作打包成一个，整合操作用的，例如上面这种图片，如果正常写是要写成3行，而且左边还都需要赋值，但是写成这样则不需要了，即用一个操作代替3个操作
- 需要注意的是，括号里面要写 [  ] ，表示一个操作序列，里面的参数都是transforms操作
# Randomcrop()的用法
- 它的功能是随机在原图片种裁剪一个h * w 大小 或者 size * size 大小的图片 
- 对应括号里要么是 （h，w） 要么是 int
# 杂谈
## 对于之后别的函数：
![](/assets/img/blog/pytorch-learning/pytorch-note-06.png)
## batchsize就理解成这一批数据集的大小
# 神经网络
## Model
- 这个是所有神经网络的骨干，就是不管你写啥网络你都得先这样去搭个框架 init 和 forward都是必须得重写的，至于图片写这样写只是一个示例，再下面一张图片就是关于这个的简单运用
![](/assets/img/blog/pytorch-learning/pytorch-note-07.png)
![](/assets/img/blog/pytorch-learning/pytorch-note-08.png)
- 然后这里注意init里面第一句用于是继承父类，然后init里一般放的就是网络有多少层啊，然后是什么层，什么卷积层，什么池化层这种
- 然后forward里面就是写前向传播
