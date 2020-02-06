---
layout:     post
title:      "Tensorflow下的数据流及其优化"
subtitle:   " \"Tensorflow下的训练数据流及其性能优化\""
date:       2019-11-10 01:01:00
author:     "HeShuai"
header-img: "img/banner/dataflow.jpg"
catalog: true
tags:
    - 深度学习
---

# Tensorflow下的数据流及其优化

### 0. 摘要
> 训练过程中高效的数据流降低可以GPU的空闲时间，加速算法验证。为最大化优化数据流程 ，结合[tensorflow官方教程](https://www.tensorflow.org/guide/data_performance)以及个人的一些使用经验，总结如下优化点：
>
> 1. 数据封装成tf推荐的tf records形式，并尽可能优化records中的样本大小，降低训练中的IO压力。
> 2. 当数据体积较大并设计比较耗时的解析操作时（如大分辨率的图片读取），分割成多个小records，可以进行IO并行读取和处理。
> 3. 采用pipeline形式，并行化CPU上的数据流准备与GPU训练过程，最大化降低GPU的idle时间。
> 4. 多线程并行化CPU上的数据流相关工作，充分利用CPU的多核心资源。
### 1. 介绍
在简单的深度学习工程中，数据与模型是基础单元。数据作为模型的输入。一般数据相关的包括IO读取与预处理，多在CPU上运行。模型训练则包括正向推理和反向梯度，在GPU上进行加速。
当数据准备与模型训练串行进行时（如下图），一次迭代时间为：
$$
T_{iteration} = T_{cpu}  + T_{gpu} \\                     
T_{cpu} = T_{io} + T_{preprocessing}
$$
![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20191110231842.png)

<center>
    Figure-1: 串行化的训练pipeline
</center>

---

上述的训练流程是很低效的，GPU的时刻处于欲求不满的状态，idle的时间取决于CPU的数据准备耗时。由于现代GPU性能比较强劲，因此CPU的瓶颈会造成极大的GPU资源浪费。比较直觉式的改进方式是采用并行化的训练流程，并行化包括两层意思：

1. CPU数据准备与GPU前后向运算的并行化。
2. CPU上的数据流程的多核心多进程并行化。

并行化之后得到如下的训练pipeline，此时单次迭代的时间为公式（2）。由于一般情况下GPU的计算时间短于CPU，因此只要CPU上的数据流程优化允许，可以取得理论的百分百GPU利用率。

$$
T_{iteration} = max(T_{cpu} , T_{gpu})
$$

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20191114132559.png)

<center>
    Figure-2:并行化的训练pipeline
</center>

### 2.  TF Data实现下的并行化训练pipeline

#### 2.1 TF Data数据准备逻辑

Tensorflow下的数据流程大致可以如图分为三个阶段：

1. 离线阶段

   该阶段为tf records的**离线**生成。tf records是tf官方推荐的文件形式，所有源数据都打包到tf records文件中，可以供后续高性能读取。

2. 数据迭代器初始化

   该阶段是数据预处理阶段，是相当重要的一个阶段，在训练过程中**在线**进行。数据流解析、数据预处理和训练准备都在该阶段完成。数据流解析是将tf records中的数据单元解析出来，如一张图片；数据预处理极为重要的阶段，模型数据预处理和数据增广都可以在此进行；训练准备则设计shuffle\batching和repeat等。

3. 在线迭代

   该步骤是在2的基础上进行迭代器准备，每一次迭代都会输出一个batch的训练data，以供训练。

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images20191111235111.png)

---

以上三个数据流步骤推荐采用[tf.data](https://www.tensorflow.org/guide/data)实现。tf.data是tf官方提供的简洁易用的数据流准备API，具有高性能和多线程并行的特点，最大化利用CPU资源，降低数据准备时间。总结，tf.data有以下特点：

1. 支持不同的数据源以及数据大小和宽度，如text、audio、image和numpy array等类型的数据。官方推荐将数据打包成tf records的形式，以最大化利用tf.data API。
2. tf records是tf官方推荐的数据中间形式，其将不同类型的数据二进制流存入一个tf record文件中，通过tf.dataset进行读取。该方式可以有效提升大批量小文件的读取效率，并方便后续基于tf.data API的高效处理。
3. tf.data采用ETL数据处理流程给，即Extract、Transform和Load。Extract指数据的IO准备；transform指数据的准备，如二进制流解析、图像语音数据的编解码。训练中最常用的数据增广也属于transform阶段；Load值将数据加载到GPU上。
4. tf.data底层实现了比较高效的多线程并行化，可以实现CPU和GPU的pipeline并行化、CPU数据处理流的多线程并行化。

#### 2. 2 数据流程的并行化

如前所述，并行化是指两方面的并行化：CPU数据准备和GPU训练的并行化；CPU数据准备的多核多线程并行化。

![](https://raw.githubusercontent.com/mightycatty/image_bed/master/images/20191114132559.png)

**CPU和GPU的并行化：**

> CPU和GPU的并行化是指数据准备和模型训练的并行化。CPU在GPU进行计算时就进行数据准备，等GPU结束立马送过去，减少GPU的idle时间。
>
> tf中的并行化通过tf.dataset.prefetch()实现

**数据加载与预处理的并行化：**

>数据在输送GPU之前会进行IO读取和预处理。其中数据增强会在线进行且很耗时，为了加快这部分的运算，tf data拉起多个线程进行IO读取和并行数据处理。tf.dataset很多api接口都会提供map_cores参数，代表并行的线程数和利用的核心数。一般来说尽可能利用所有的可能CPU核心。

#### 2.3 Sample Code

```python
def input_fn_from_tfrecord(tfrecord_filenames_list, tf_example_parser_fn, batch_size=32, map_cores=4, augmentation=False, aug_fn=None, debug=False, shuffle_buffer_size=1000, **kwargs):
    """
    simple input_fn consuming tf records
    :param tfrecord_filenames_list: dir to a tf record of list of which
    :param tf_example_parser_fn: fn for parse tf sample from tf record
    :param batch_size: batch size
    :param map_cores: num of corse for parallism
    :param augmentation: augmentation flag
    :param aug_fn: augmentation fn
    :param debug: debug flag
    :param shuffle_buffer_size:
    :param kwargs:
    :return:
    """
    if type(tfrecord_filenames_list) is str:
        tfrecord_filenames_list = [tfrecord_filenames_list]
    # tf records io读取，支持多个同时读取或者一个多线程读取，num_parallel_reads指拉起的io线程数，buffer_size是缓冲池（MB）
    num_parallel_reads = len(tfrecord_filenames_list)
    dataset = tf.data.TFRecordDataset(tfrecord_filenames_list, num_parallel_reads=num_parallel_reads,
                                      buffer_size=None)
    dataset = dataset.prefetch(buffer_size=batch_size * 2)
    # shuffle需要谨慎的操作，buffer_size是shuffle池大小。太小了shuffle意义小，大了性能很差，推荐生成records的时候就shuffle
    dataset = dataset.shuffle(buffer_size=shuffle_buffer_size)
    # repeat, tf records循环读取，无穷无尽，为训练准备
    dataset = dataset.repeat()
    # 传入对应你生成tfrecords时的parser函数
    dataset = dataset.map(tf_example_parser_fn, num_parallel_calls=map_cores)
    # 数据预处理或者增强函数，通过map api实现。num_parallel_calls控制核心数，二话不说有几核用几核就是了
    if augmentation:
        assert aug_fn is not None, 'augmentation function must be provided if augmentation flag is true'
        dataset = dataset.map(aug_fn, num_parallel_calls=map_cores)
    # 训练的batching准备，相当于前面的所有sample都堆成一个batch再送GPU
    dataset = dataset.batch(batch_size)
    # 顾名思义，for debug
    if debug:
        iterator = dataset.make_one_shot_iterator()
        dataset = iterator.get_next()
    return dataset
```



### 3. 优化图像数据源的tf records生成

TF records是TF官方的御用数据格式。其是一种中间格式，不同的源数据（text\语音\图像）统一打包成这个格式，供后续的tf.data API使用。但是tf records也不是万能的。推荐**数据相对稳定的时候使用**，而快速的验证则采用其他更快捷的形式：

> Pros: 大批量数据整合成一个文件，高效IO；作为御用格式可以最大化发挥tf.data的性能
>
> Cons: 生成贼鸡儿麻烦（动不动个把小时）；每次更新数据都需要重新生成，不利于小规模数据验证；由于tf runtime没有exception机制，tf records里有一个数据坏了，整个就废了；不利于快速方便的可视化。（这么一列好像还挺蛋疼的，为啥要用呢:sweat:）



([官网食用指南](
https://www.tensorflow.org/tutorials/load_data/tf_records))，主要涉及tf.train.Feature和tf.train.Example两个类。

>tf.train.Feature: 定义了训练中的数据（即所谓feature）格式，支持int、byte以及float。如image就可以作为一个Feature实例。

>tf.train.Example: 定义了训练中一个完整的sample，sample内包含单个或者多个Feature。如图像分类任务中，一个sample包含了**图像+标签**两个Feature。当然sample实例内可以存在其他不参与训练的Feature，如图片的长宽等。

下图是TF record的简要生成流程：
> 1. Raw data按照选定的格式封装成一个或者多个Feature实例。
> 2. tf.train.Example结构一个Features字典，生成一个可用于训练的完整数据单元。
> 3. 通过tf.python_io.TFRecordWriter将一个或者多个（serialized）Examples写入.record文件中。

### 4. 总结

以下是全文的一些关键点总结：

1. 数据相对稳定的时候推荐先生成tf records，如果还在快速迭代或者更换过程，还是python迭代器比较方便。
2. tf records生成的时候尽可能降低tf records中每个sample的大小，如图片分辨率和大小，最终降低整个tf records的大小，实现更快的io读取。
3. 不要把tfrecords当作一种数据存储手段，而应该是训练手段，目标是能最大限度加速训练过程。一些固定的操作，如固定的预处理或者类型转换都在离线生成tfrecords时一次性生成，避免训练迭代中重复计算。
4. 尽可能在数据生成tf data的时候就shuffle，尽量不要在线shuffle。一个是会伪shuffle，另外分割train/val/test的时候会造成混乱（每次都会分割不一样，当然也可以先分割再shuffle来避免）。
5. tf.dataset.prefeatch用起来，CPU和GPU并行的关键步骤。
6. 凡是tf API中带map_cores的，不要吝啬，多核用起来:smile:。
7. 鉴于tf.data.map如果调用原生python是走的eager模式，会有一定的性能问题，推荐尽可能的采用tf api写数据预处理函数（虽然真的很垃圾很难用）。

todo:

1. 预处理https://www.tensorflow.org/guide/data#preprocessing_data



