# Paper Reading

[TOC]

---

## [Learning Efficient Convolutional Networks through Network Slimming-2017](https://arxiv.org/pdf/1708.06519.pdf)

### Abstract

通过在BN的$$\gamma$$上增加L1约束在训练阶段达成卷积通道层次的稀疏化(sparsity)，并以$$\gamma$$作为ranking指标，后续裁剪掉一些通道，以实现模型压缩。方法相对容易实现，代码量较少另外从作者的数据结果看似乎很阔以。

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200215181620.png)

### Details

1. 利用多轮裁剪的方式逐步降低模型的体量。

2. 针对非VGG等straightforward的模型，需要稍微特殊处理。

3. bn上l1似乎还有全局正则化的作用，相对原来的模型带来了表现提升。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20200215181552.png)
4. 裁剪后会有精度损失，需要finetune之后才会带来提升。

### Personal Thoughts

1. 增加L1约束是不是增加了总的正则化力度，对于本身模型容量就不够（欠拟）的是否会损失性能。

### Reference



