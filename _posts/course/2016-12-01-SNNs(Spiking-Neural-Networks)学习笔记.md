---
layout: post
title: SNNs(Spiking Neural Networks)学习笔记
category: course
keywords: 神经与认知计算
---

这次学习SNNs主要是参考了以下这篇文章：

    Diehl P U, Neil D, Binas J, et al. Fast-classifying, high-accuracy spiking deep networks through weight and threshold balancing[C]// International Joint Conference on Neural Networks. IEEE, 2015.
    
以及以下这份Code：

    http://github.com/dannyneil/spiking_relu_conversion


- 这篇文章使用的数据集是mnist手写数据集。
- 文章通过将FCNs（Fully-Connected Networks）转换成SNNs的方式来构造SNNs网络。

### FCNs：
```graphLR
    A[input:784*1] --> |weights:784*1200| B[fc_1:1200*1] 
    B --> |weights:1200*1200| C[fc_2:1200*1] 
    C --> |weights:1200*10| D[output:10*1]

```
### SNNs

首先是输入的图片转化为刺激信号：
```graphLR
    A0[input:784*1] --> |spike_snapshot | A[spike_input:784*1]
```
spike_snapshot的刺激方法为：
```matlab
spike_snapshot = rand[784,1] * 5;
spike = input >= spike_snapshot;
```
即对输入input进行随机刺激，并让那些原本值比较大的神经元发出信号（spike_input），信号的值是0/1。

---
以spike_input作为输入层，通过全链接来刺激第二层1200个神经元，并使第二层的神经元发出一个spike来刺激下一层，以此类推。以下以输入层刺激第二层作为例子：
```graphLR
    A[spike_input:784*1] --> |乘以权重weights:740*1200并加在fc1的膜电位上| B0[mem_fc1:1200*1]
```
```graphLR
    B0[mem_fc1:1200*1] -->|如果膜电位mem_fc1>=threshold电位,发出信号并置电位为0| B[spike_fc1:1200*1]
```
`spike_fc1`就是`fc1`层发出的刺激信号，用于刺激下一层。同时，`fc1`层还要记录下1200个神经元的发出信号时间t（如果发信号了的话），用于更新生命周期内的最后一次发信号时刻，代码里为`refrac_end`。以及每个神经元在整个声明周期内的总发信号数量，代码里为`sum_spikes`。

从这里可以看出，随着`t = dt:dt:opts.duration`的时间增长，第一层神经元是不断rand出信号来刺激，后面几层的神经元膜电位不断增加并超出threshold被置0，同时发出信号刺激下一层。

在这期间，每层的每个神经元有三个全局属性（在生命周期内不断变化）：

- 膜电位（mem）：除了第一层输入层之外都有的属性，在证明周期内逐渐积累变大并被不断置0，同时发出信号。
- 最后一次发信号的时间（refrac_end）：在生命周期内不断被更新。
- 每个神经元在生命周期内发信号的总数（sum_spikes）:在声明周期内不断累加。

---
最后通过output层（第四层）的sum_spikes：10*1来判断预测种类：
```matlab
[~, guess_idx] = max(nn.layers{end}.sum_spikes');
```

