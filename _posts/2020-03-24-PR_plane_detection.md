---
layout:     post
title:      "PaperReading-平面检测"
subtitle:   " \"PaperReading-平面检测\""
date:       2020-03-24 22:02:00
author:     "HeShuai"
header-img: "img/banner/post-default.jpg"
catalog: true
tags:
    - 推理
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


# Plane Detection & Construction

平面模型：
$$
nx = d \\
n: normal vertor平面法向量 \\
d: offset, 坐标原点到平面的距离
$$
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200307111230.png)

从2D图像的平面估计拆分开就变成了三个子问题：
1. 实例分割：由于存在多个平面，所以是一个2类多实例分割问题
2. 回归问题（需要求解四个标量参数，法向量 n = (x, y, z)和偏置量d）
	- n求解类似实例分割的边框回归，具有实例local的特点
	- d的求解从深度估计中得到，假设为z：则（实际上求某平面所有像素位置的偏置量平均）
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200307152050.png)

## Paper List

### Constructed from RGB Image

#### 1. [(2019)PlaneRCNN: 3D Plane Detection and Reconstruction from a Single Image](https://arxiv.org/pdf/1812.04072.pdf)
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200307152328.png)

**概要**
文章采用maskRCNN类似的思路来处理平面检测问题，其中分割分支针对平面像素分割和深度估计，平面参数回归则采用边框回归类似的思路。其中nx=d中的偏置量并非直接回归，而是经过法向量和深度估计求得（需要知道相机内参）。
模型有三个heads：

- 实例分割支：local地分割每个平面像素点
- 深度估计支：全局估计图像深度，类似于语义分割
- 参数回归分支：采用了与anchor类似的思想，并没有直接回归法向量，而是在一组法向量基元的基础上回归offset。
主干模型（Plane detection network后面还有一个segmentation refinement network），用以优化分割结果。

**ToRead:**

- 传统3D平面检测算法 [10, 12, 37, 38, 52]

#### Code



#### 2. [(2018)PlaneNet: Piece-wise Planar Reconstruction from a Single RGB Image](https://arxiv.org/pdf/1804.06278.pdf)
**概要**：
	基于分割模型的一个平面检测算法，稠密地检测多个平面实体的分割mark和平面参数。模型有三个分支：平面参数预测分支、平面pixel实例分割分支和深度估计分割。前者为参数回归任务，后两者为分空间分类/回归任务。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20200307110845.png)

疑问：
1. 分割/深度预测分支没有类似maskRCNN的ROI，多实例分割采用固定数目的分类形式，是否不太妥当或者增加无效计算量。

#### 3. (2018)[Recovering 3D Planes from a Single Image via Convolutional Neural Networks](https://faculty.ist.psu.edu/zzhou/paper/ECCV18-plane.pdf)

#### 4. [(2019)Single-Image Piece-wise Planar 3D Reconstruction via Associative Embedding](https://arxiv.org/pdf/1902.09777.pdf)

    To Read
    [6, 2, 24, 32, 15, 10, 12].
---

To read List:
- Single-view 3d scene parsing by attributed grammar
- Lifting 3d manhattan lines from a single image
- Unfolding an indoor origami world
- Dilated residual networks