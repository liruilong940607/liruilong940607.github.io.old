# SemanticFusion阅读笔记 by@Yang-Sheng

标签（空格分隔）： SemanticFusion

---

# 目录
[TOC]

## 本节内容
SemanticFusion阅读笔记，实现细节Mark等。

# 提到的相关工作
- SLAM++。
- RDF，文中出现多次的[8]: Dense 3d semantic mapping of indoor scenes from rgb-d images, ICRA, 2014, 待查。
- 使用RGB, Depth, **Normal**作为输入进行CRF或CNN。

# 流程及Components
- ElasticFusion，实时SLAM，Surfel点云。
- CNN，处理单帧RGB-Depth-Normal，得到每个像素关于每个类别的概率。
- Bayesian update schemme，维护每个Surfel的Class probability distribution。
- CRF regularisation scheme，利用几何性质提升结果。

##　ElasticFusion
这个反正忘不了，四组关键词：
- Surfels点云，Deformation Graph操纵
- ICP+RGB的F2F Trans
- 局部回环(基于时间窗)
- 全局回环(基于关键帧)

然而这篇文章用了8m的Depth Cutoff，ElasticFusion本身为3m！

## CNN
基于Caffe，Deconvolutional Semantic Segmentation：《Learning deconvolution network for semantic segmentation》，这篇本身是一个RGB-CNN，没有带Depth玩。但是本文中把Depth作为第四个Channel送了进去，并且直接从0~8m打到0~255。
图片分辨率变为224x224送CNN，输出回到480P应用到Surfels。
每10帧算1帧，最终速度有25.3Hz，每帧都算则只剩下8.3Hz。

### CNN Training
对CNN核心内容不是太懂，这里简单翻译(抄)一下：
用在Pascal Voc12上训的[Pretrained Model](https://github.com/HyeonwooNoh/DeconvNet)，然后finetune on NYUv2，13类(TABLE II)。

### CNN Testing
他们自己标了一些Surfels点云的场景(一个Office Room)。

## Incremental Semantic Fusion
每一个新的Surfel拿到一个Uniform Distribution。对于每一个新帧，乘以CNN算出的当前帧的概率值，再归一化。

## Map Regularisation
跑Graph-Cut：Applying a Fully-connected CRF with Gaussian edge potentials to surfels，公式倒是直接贴在Paper上了。
每500帧算1次，能稍微提高准确率，在线处理的时候可以考虑Disable掉或者在最后用一次。

疑问：Fully-Connected Graph去解Graph-Cut(Dense CRF)能实时么？
据说[13]是干这一行的：Efficient Inference in Fully Connected CRFs with Gaussian Edge Potentials, NIPS, 2011.到时候拿他们[实现](http://graphics.stanford.edu/projects/densecrf/)怼一下试试。


# 总结
从他们结果上来看，10帧, 500帧的降频没有对准确率造成太大影响，维持在50%左右。用的CNN还是Deconv，做Semantic Labeling肯定是这一套。最后的GraphCut需要自己再去读读[13]，比较一下我以前做分割的时候用的Patch-based Graph-Cut和这个的速度、效果差异。



