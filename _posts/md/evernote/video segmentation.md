# Paper Reading

[TOC]

# Interactive Video Segmentation

## 1. [Fast User-Guided Video Object Segmentation by Interaction-and-Propagation Networks](https://arxiv.org/pdf/1904.09791.pdf)

### Abstract

18年DAVIS交互式分割的冠军，采用双支分割的形式：第一个Unet负责交互帧的分割；第二个则将交互帧的结果传递到其他帧。交互Unet的中间特征进行某种累积操作，提供给帧传递模型。

### Idea Details

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200213130821.png)

### Training & Implementation Details

1. [源码](https://github.com/seoungwugoh/ivs-demo)：ubuntu下的可视化demo代码，没有训练代码
2. 有一定的储存开销，每一帧都需要储存上一轮的分割结果，以及需要储存一份中间特征。

---

## 2. [Interactive Image Segmentation via Backpropagating Refinement Scheme](https://vcg.seas.harvard.edu/publications/interactive-image-segmentation-via-backpropagating-refinement-scheme/paper)

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
2. 

### Personal Ideas

1. 该种方法不修改原模型参数，更像一个在以模型全局求解为基础上的针对某个样本的局部求解。视频流的分割是否也能用这种方法？
2. 这种在线迭代算法的效率？

### Reference

[1] [sample video](https://youtu.be/KOcpzBAVfFE)

[2] [source code](https://github.com/wdjang/BRS-Interactive_segmentation)，非完整代码，model建于caffe。

## 3. [Video Object Segmentation using Space-Time Memory Networks](https://arxiv.org/pdf/1904.00607.pdf)

### 

---

# Video Segmentation



### Abstract



