# Paper Reading

## Training相关

### 1. [Rethinking ImageNet Pre-training](https://arxiv.org/pdf/1811.08883.pdf)

凯明大神18年的文章，主要论点和结论：
> 1. 在充足数据的情况下，pretrain于imageNet对最终结果的精度影响不大。
> 2. 严重不充足训练数据pretrain于imageNet可以显著提升精度，另外数据数量不仅仅跟图张有关，跟instance、pixel也有关。
> 3. 预训练可以加快模型收敛速度。
> 4. 针对于空间精度有关的任务，如分割和detection，预训练的作用更小。

其他一些有意思的点：

> 1. （目标检测）不同模型和训练数据下超参数的选择对结果影响很大。
>   ![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20191031203129.png)
>
> 2. 大学习率训练较长的时间然后在快结束的时候转小学习率（/10）。如果以小学习率训练一段时间会造成过拟合。

疑问：

> 1. 在imageNet上的classification pretrain对detection和instance segmentation没用，但是在同任务在开源数据上finetune是否有影响。

### 2. [Understand Batch Normalization](https://arxiv.org/pdf/1806.02375.pdf)

细节点：
> 1. 浅模型允许更大的learning rate，作者认为前面的波动在深模型后面会被放大。

### 3. [Adaptive Graadient Methods With Dynamic Bound of Learning Rate](https://openreview.net/pdf?id=Bkg3g2R9FX)
文章认为：
1. 相对于SGD，以Adam为代表的adaptive opt虽然能让training数据看起来很好看，却有着更差的泛化能力（on unseen data）。
2. 作者认为这是由于动态lr的设置使得训练过程中存在极大（1000）和极小（0.001）的lr，使得模型收敛到一个泛化能力较大的局部最优点上。


## Model Design


### 1. MobileNetv3
主要创新和贡献点：
> 1. 提出HS激活函数，以较小的计算代价逼近sigmoid的非线性能力，增强特征的非线性变化。
> 2. 在backbone里增加类似通道注意力的模块。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20191031235159.png)
> 3. 针对分割任务设计了一个带注意力机制的轻量级分割头部
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20191031235325.png)

个人理解和观点：
> 1. 模型结构设计中，expansion的意义在于升维，增加特征的非线性。如在低纬度不可分的映射到高维增强显性可分能力。
> 2. expansion增加线性可分之后，采用channeL注意力似乎可以进行特征选择，从而进一步提升线性可分能力。
> 3. 层度越深的地方同一分辨率下卷积串行操作更多，原因可能在于金字塔中要求高层塔具备低分辨率高语义特点，更多的串行卷积可以加强这两个特点。
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20191031235422.png)