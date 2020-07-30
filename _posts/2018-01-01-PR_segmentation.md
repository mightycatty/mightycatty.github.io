---
layout:     post
title:      "PaperReading-Segmentation"
subtitle:   " \"Paper Reading: Segmentation\""
date:       2018-01-01 01:01:00
author:     "HeShuai"
header-img: "img/banner/dataflow.jpg"
catalog: true
tags:
    - Paper Reading
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

[TOC]

# Paper Reading

# Interactive Video Segmentation

## 1. (2018)[Fast User-Guided Video Object Segmentation by Interaction-and-Propagation Networks](https://arxiv.org/pdf/1904.09791.pdf)

### Abstract

18年DAVIS交互式分割的冠军，采用双支分割的形式：第一个Unet负责交互帧的分割；第二个则将交互帧的结果传递到其他帧。交互Unet的中间特征进行某种累积操作，提供给帧传递模型。

### Idea Details

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200213130821.png)

### Training & Implementation Details

1. [源码](https://github.com/seoungwugoh/ivs-demo)：ubuntu下的可视化demo代码，没有训练代码
2. 有一定的储存开销，每一帧都需要储存上一轮的分割结果，以及需要储存一份中间特征。
3. 作者开源的demo实际效果一般，交互次数太多。误分割的地方很顽固，基本每一帧都需要人工矫正，个人怀疑是帧间传递能力不足。

---

## 2. (2019)[Interactive Image Segmentation via Backpropagating Refinement Scheme](https://vcg.seas.harvard.edu/publications/interactive-image-segmentation-via-backpropagating-refinement-scheme/paper)

### Abstract

针对静态图像的点式交互分割，核心创新点在于采用BRS(backpropagating refinement)进行反馈优化。BRS采用迭代式能量优化的方式，优化输出的interaction map，使模型的输出不误分割用户给定的点（为优化函数的一项）。注意，该种方法无需重训或者修改已经寻好的通用分割模型，迭代优化的是输入，以优化模型输出，因此可以有更广泛的用途。但是个人很怀疑这种在线迭代的效率，论文中也没有提及。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200215110334.png)

### Details
1. 正向就是一个常规的分割模型，但是输出的原图和交互图。交互图从用户输入的点生成的一个能量图，能量分布与用户的点的距离有关，似乎有点像人工势能图（见上图的输入）。
2. 分割模型的具体结构如图，UNet、DenseNet,、squeeze and excitement、二阶段式，refine decoder接受image+coarse mask输入。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200215110314.png)
3. BRS采用在线迭代能量优化方式，优化的是神经网络的输入，而不修改模型本身参数。本文中优化函数有两项：用户交互能量图与神经网络模型生成分割结果的距离、用户交互图的变化约束（防止大幅度的变化）。第一项的梯度似乎需要通过对整个神经网络求导。优化算法为BFGS算法。

![1581741248129](C:\Users\herschel\AppData\Roaming\Typora\typora-user-images\1581741248129.png)

### Training & Implementation Detail

1. 训练数据采用SBD dataset(8498)，所有的用户点击都是模拟生成的。生成细节论文有写。

   

### Personal Ideas

1. 该种方法不修改原模型参数，更像一个在以模型全局求解为基础上的针对某个样本的局部求解。视频流的分割是否也能用这种方法？
2. 这种在线迭代算法的效率？

### Reference

[1] [sample video](https://youtu.be/KOcpzBAVfFE)

[2] [source code](https://github.com/wdjang/BRS-Interactive_segmentation)，非完整代码，model建于caffe。

## 3. (2019)[Video Object Segmentation using Space-Time Memory Networks](https://arxiv.org/pdf/1904.00607.pdf)

## 4. (2020)[SINet: Extreme Lightweight Portrait Segmentation Networks with Spatial Squeeze Modules and Information Blocking Decoder](https://arxiv.org/pdf/1911.09099.pdf)

端上高实时portrait分割网络，主要创新点有两个：information blocking和spatial squeeze block。前者控制高低语义信息流，后者类似一种加速trick，低分辨率上做完卷积再缩放回去。
- Information Blocking
作者认为语义特征上高置信度区域不需要引入shortcut低频信息，因此设计了一个blocking模型来防止shortcut中的信息干扰。类似反向attention操作（1 - sigmoid(x)）
- Spatial Squeeze Module(S2-module)
双线性差值到低分辨率做完卷积再差值回去，此时卷积大小为1也具有一定的空间特征提取能力。并采用了shufflenet的思想，增强信息流动性。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200427195858.png)

## 5. (2018)[Learning a Discriminative Feature Network for Semantic Segmentation](https://arxiv.org/pdf/1804.09337.pdf)

## 6.(2018)[Pyramid Attention Network for Semantic Segmentation](https://arxiv.org/pdf/1805.10180.pdf)
文章重点从高低语义信息融合和多分辨率特征方向入手，提出Global Attention Upsample module融合高低语义信息、Feature Pyramid Attention来获取不同spatial分辨率的特征金字塔。Attension思想贯穿两点。
- Feature Pyramid Attention
整体思路还是获取特征层面的多分辨率金字塔，但是采用了Spatial attention的思路，操作上可理解维特征平面grid的attention。相对ASPP或者PSP操作精细很多，但是这么操作不知道在实时性里面会引入多少计算量。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200428150136.png)
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200428151018.png)
- Global Attention Upsample
作者argue naive的upsample是不行的，可以利用高层次语义特征对低层次特征进行channel attention。疑问是high-level features与low-level features不存在直接的信息交流的情况下是如何产生attention vector的。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200428150758.png)
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200428151042.png)
## 7.(2019)[FastFCN: Rethinking Dilated Convolution in the Backbone for Semantic Segmentation]
设计了一个联合上采用模块加强FCN的特征，可以得到dilatedCNN同等的效果但是快数倍。问题是看起来结构也很复杂, 是否适合端上.
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200516151751.png)
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200516151945.png)
## 8.(2019)[https://arxiv.org/pdf/1910.08711.pdf]
SSL loss, mask的structure的相似度. 另外调的参数有点多.
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200521165116.png)

## 9. (2018) Learning to Predict Crisp Boundarie

提出Dice Loss来应对boudary这种极端不平衡的情况, Loss维DICE coefficient的倒数, 描述的是两个集合的相似程度, 当两个集合完全一致时, 为1. 该Loss是一种全局性的考虑, 结合CE等pixel level的loss可以获得一些不错的结果. 
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200522164938.png)

## 10.(2020)[Joint Semantic Segmentation and Boundary Detection using Iterative Pyramid Contexts](https://arxiv.org/pdf/2004.07684.pdf)

模型同时估计mask和semantic boundary，通过loss同时束缚mask和boundary来达到提升mask的目的。

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200526151754.png)

## 11.(2018)ExFuse: Enhancing Feature Fusion for Semantic Segmentation
文章主要探索如何更好地融合分割任务中的高低层次语义特征

## 12.(2019) [Decoders Matter for Semantic Segmentation: Data-Dependent Decoding Enables Flexible Feature Aggregation](https://arxiv.org/pdf/1903.02120.pdf)
作者提出一种可以实现任意上采样倍数且轻量级的decoder以取代DLV3+的decoder，大体思想就是depth to spatial，类似subpixel的思想。
模仿实现了一下，发现得到的边缘圆润度很差，棋盘效应更加严重。在原论文和githu的一份pytorch中看到相似的可视化结果。

## 13.(2020)[]