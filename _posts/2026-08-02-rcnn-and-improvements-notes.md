---
layout: blog_post
title: "R-CNN 及后续改进学习"
date: 2026-08-02
tags:
  - R-CNN
  - 目标检测
  - 深度学习
---
[学习素材](https://www.bilibili.com/video/BV1af4y1m7iL?spm_id_from=333.788.videopod.episodes&vd_source=ea024c91ade3352ca6e239450cb32fcf)
# R-CNN 及后续改进学习
## 算法流程
- 一张图像生成1K~2K个候选区域戈(使用Selective Search方法)
- 对每个候选区域，使用深度网络提取特征
- 特征送入每一类的SVM 分类器，判别是否属于该类
- 使用回归器精细修正候选框位置
![R-CNN 算法流程](/assets/img/blog/rcnn-learning/rcnn-overview.webp)

> Selective Search方法是什么？

以前传统的目标检测采用的滑动窗口，即使用一个窗口去移动，看每一块区域是不是我的目标对象
而Selective Search 则是先进行 图像分割 将图像划分成很多小区域
然后计算小区域间的相似度（当然如何判断相似有以下4个因素）
颜色相似、纹理相似、大小相似、形状相似
然后再将区域相似度较高的不断合并

> 那为什么会有 1k ~ 2k 个框呢？

因为每次合并的过程都被保留下来了，类似于下面这个过程
![Selective Search 区域合并过程](/assets/img/blog/rcnn-learning/selective-search-process.webp)

> 深度网络提取特征，采用的是哪个网络？为什么采用？

采用的是AlexNet CNN网络，但是去掉了全连接层，因为这里主要是为了进行特征提取
因为AlexNet CNN以及在image Net上训练过了，所以它以及有了很多图像的通用视觉特征

注意每一个SVM分类器都是一个二分类的，即是或不是
![R-CNN 的 SVM 分类器](/assets/img/blog/rcnn-learning/rcnn-svm-classifiers.webp)

其中NMS（Non-Maximum Suppression）就是非极大值抑制

回归器的详细理解留到faster-RCC了

# Fast-RCNN
## 算法步骤
- 一张图像生成1K~2K个候选区域戈(使用Selective Search方法)
- 将图像输入网络得到相应的特征图，将SS算法生成的候选框投影到特征图上获得相应的特征矩阵
- 将每个特征矩阵通过ROI pooling层缩放到7x7大小的特征图，接着将特征图展平通过一系列全连接层得到预测结果
![Fast R-CNN 算法流程](/assets/img/blog/rcnn-learning/fast-rcnn-workflow.webp)
> ROI是啥？

ROI 是 region of interest，也就是感兴趣区域

# Faster-RCNN
## 算法步骤
- 将图像输入网络得到相应的特征图
- 使用RPN结构生成候选框，将RPN生成的候选框投影到特征图上获得相应的特征矩阵
- 将每个特征矩阵通过ROI pooling层缩放到7x7大小的特征图，接着将特征图展平通过一系列全连接层得到预测结果
![Faster R-CNN 算法流程](/assets/img/blog/rcnn-learning/faster-rcnn-workflow.webp)
Faster-RCNN 可以 简单理解成 RPN + Fast-RCNN
也就是使用RPN去代替SS算法

>RPN是啥，以及RPN网络架构是咋样的？以及它的流程？

RPN是Region Proposal Network也叫区域候选网络

网络结构：
![RPN 网络结构](/assets/img/blog/rcnn-learning/rpn-architecture.webp)
首先对于特征图，使用一个3x3的滑动窗口进行滑动，每滑动到一个位置，就生成一个一维的向量，至于这个一维的向量有多大，就依据所使用的骨干网络，例如使用VGG16就是512，再通过两个全连接层对这个一维向量得出类别概率以及回归框参数

>为什么是2K呢？

首先K是因为有K个anchor box，2K是区分它是前景还是背景，也就是它是真正的物体还是背景
但是注意这里的2K中只是区分它是背景还是前景，并没有分类

对于特征图上的每个3x3的滑动窗口，计算出滑动窗口中心点对应原始图像上的中心点，并计算出k个anchor boxes(注意和proposal的差异)
![Anchor boxes 示意图](/assets/img/blog/rcnn-learning/anchor-boxes.webp)

>那这里的anchor box的大小以及尺寸如何定呢？

这个就是根据经验了，你比如原论文中就给了 128 256 512 三种，以及对对应的比例1:1,1:2,2:1
后续我们在double head-RCNN中也对这个进行了改进，因为我们做的是航拍图像小目标检测，它和常规的肯定不一样

注意anchor 和 候选框不是一个东西，我们是根据前面输出的4K个参数（边界框回归参数）将anchor调整到候选框

>那如何定义是前景还是背景？

前景：
- 所有的anchor中与真实目标框的iou最大的那个是前景
- anchor与真实目标框的iou大于0.7的是前景
背景
- anchor与所有的真实目标框的iou都小于0.3的是背景

需要注意的是RPN生成的anchor也是要训练的，就是它需要去筛去一部分不需要的anchor，并且也需要去调整到候选框的位置
所以你能在算法步骤图中看到两个classification loss
所以整个的过程可以这样总结
我对于每个位置我都生成了一系列的anchor，然后我去训练我哪些anchor是要保留的
然后训练好了，我的anchor也就变成候选框
然后将这些候选框对于的特征矩阵再通过ROI pooling变成7x7，然后再进行分类（这部分就是Fast-RCNN的功能）

![Faster R-CNN 训练方法](/assets/img/blog/rcnn-learning/faster-rcnn-training.webp)

## Double Hear R-CNN
Double Head RCNN 不是 RCNN 系列的官方第四代演进（不像 Fast/Faster RCNN），它是针对 Faster RCNN 检测头（Head）部分的一种改进方法。
它主要解决一个问题：分类（Classification）和定位（Localization）这两个任务的特征需求不同，但是传统 Faster RCNN 却强迫它们共享同一个检测头。
所以 Double Head RCNN 的核心思想：设计两个不同的 Head：一个负责分类，一个负责边界框回归，让两个任务分别学习适合自己的特征。

> 那它做了哪些改进呢？

以前是将ROI Pooling之后的结果送入共享全连接层，虽然我们在fast-RCNN中看到最后是使用了两个FC来输出
但是这两个FC使用的是同一个特征，就是他们两共享了同一个特征

而Double Head 则没有共享
它将ROI 之后的特征送入分类头（FC，全连接层）做分类
然后还将特征送入回归头（Conv，卷积层）做回归

> 那为什么这么做有用

因为分类注意的是语义信息，也就是是或不是的问题
而回归注意的是在哪的问题，是空间信息

使用分类用了FC全连接层，它可以很好的用于分类
而回归则使用了卷积层，因为它可以很好的关注到目标在整张图中的位置信息

### 以航拍小目标图像检测为例讲解Double Head RCNN的应用
>什么是ResNet-50

它是现阶段，Faster-RCNN中主流使用的一个骨干网络，就和上面使用VGG16，ZF一样

>为什么在ResNet-50中加入Transformer？

因为Transformer的注意力机制可以更好的关注关键区域，从而增加小目标的语义表达
并且我们这里是将Transformer作为一个黑盒来使用的，所以并未对其内部架构有很深的理解

>为什么引入DCN可变形卷积，以及什么是DCN？

因为普通的卷积核是固定的，都是nxn，但是航拍图像，它会受到拍摄角度的变化，原本的车会变形，所以使用DCN可以对图像的感受更灵活，对尺度的变化，形变都能刚好的感受

DCN：可变形卷积，也是让传统的3x3卷积核，变成不规则形状的卷积核

>什么是CARAFE-FPN？

首先FPN是传统的多尺度特征融合，但是传统的有一个缺点，它在上采样的时候通常采用双线性插值，它只选择附近的，邻近的。但是它对内容的理解是不够的
而ontent-Aware ReAssembly of FEatures，它是内容感知的特征重组，这样可以让我们的上采样过程不丢失过程细节，也不会引入过多的噪点，增强了小目标区域的特征表达

>Anchor的尺度是如何选择的？

我们是根据VisDrone-DET2021数据集中的标注框的分布来确定的
最终选择了20、40、80、140、210
以及高宽比0.65、1.2、2.4 这样15种不同的anchor
