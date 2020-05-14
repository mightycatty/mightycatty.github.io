---
layout:     post
title:      "PaperReading-Model Design"
subtitle:   " \"PaperReading-Model Design\""
date:       2018-01-01 22:02:00
author:     "HeShuai"
header-img: "img/banner/post-default.jpg"
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

# Paper Reading - Model Design

## Paper List

#### 1. [(2018)ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design](https://arxiv.org/pdf/1807.11164.pdf)

FLOPs只能大概说明CONV部分的开销，却无法说明其他的开销，如内存读取和并行化等。
- 点卷积输入输出通道同时内存读取MAC开销最小
- 过度的分组卷积虽然控制了FLOPs，但是内存开销会显著增加
- 模型非线性（分叉）不利于并行化，（extra overheads such as kernel launching and synchronization）
- Element-wise操作同样值得注意（activation/add/depthwise conv）

因此shufflenetv2的设计指导：

1. 卷积输入输出通道一致（降低内存读取开销）
2. 控制分组卷积造成的内存开销
3. 降低模型分叉，尽可能线性
4. 减少element-wise操作

相对v1的改进：

1. 去除分组卷积
2. 利用channel split + channel shuffle增加双branch之间的信息融合和reuse
3. 去除element-wise add，concat替代

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200415104402.png)

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200415103113.png)

#### 2. (2019) [Searching for MobileNetV3](https://arxiv.org/pdf/1905.02244.pdf)

**要点：**

大体思路就是人工设计基于SE+HSWISH的base block，NAS搜索大体框架，然后人工对expensive layers进行调整的方式得到的mobilenetv3。相对v2，v3 small更精更快。另外针对分割任务提出了一个LR-ASPP结构（Lite Reduced Atrous Spatial Pyramid Pooling）。


- 在mobilenetv2的基础上增加SE结构，类似channel上的attention

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200415123339.png)

- Hard-swish，增加模型的非线性能力，但是会带来性能损耗。mobilenetv3-small中仅在第一层和最后的一些block中使用

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200426143535.png)
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200426143556.png)

- 提出新的轻量分割头部模块
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200426143958.png)

- Mobilenetv3 Large

![1586952123190](C:\Users\heshuai\AppData\Roaming\Typora\typora-user-images\1586952123190.png)

- Mobilenetv3 Small

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200415123239.png)

#### 3. (2019) [GhostNet: More Features from Cheap Operations](https://arxiv.org/abs/1911.11907)

**要点：**

论文基于卷积特征有冗余的特点，提出计算量更少的ghost模块来生成（逼近）这些冗余的特征图谱。

- Ghost Module

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200426174928.png)

- Ghost Block
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200429131934.png)
- GhostNet
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200426174548.png)

#### 4. (shufflenet)[https://arxiv.org/pdf/1707.01083.pdf]

#### 5. (ESPNetv2)[]