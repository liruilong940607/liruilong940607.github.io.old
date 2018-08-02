---
layout: post
title: Paper笔记：SemanticFusion
category: lab
keywords: segmentation caffe slam
---

## 1. 文章相关

**SemanticFusion: Dense 3D Semantic Mapping with Convolutional Neural Networks**[[PDF]](https://arxiv.org/pdf/1609.05130v2.pdf)

![屏幕快照 2016-11-11 下午9.42.28.png-717kB][1]


    B. CNN Architecture
        基于caffe，采用Noh et. al.的 Deconvolutional  Semantic  Segmentation 网络结构，Deconv-S-S网络基于VGG16，但是增加了额外的max unpooling和deconvolutional layers。Deconv-S-S网络是基于RGB的输入，本文在此基础上调整代码使其接受4通道的输入
        nyudv2只有795张标注了训练图片，所以the  depth  modality  lacks the large scale training datasets of its RGB counterpart.文章initialized the depth filters with the  average  intensity  of  the  other  three  inputs。and  converted  it from  the  0–255  colour  range  to  the  0–8m  depth  range  by increasing the weights by a factor of 32
        图片resize到224*224后进入CNN（using  bilinear  interpolation  for  RGB, and  nearest  neighbour  for  depth），用Eigen et. al.实现的代码时，是resize到320*240.输出时上采样到640*480（nearest neighbour）给surfels
        
    A. Network Training 
        用Noh et.al.（在PASCAL VOC2012上训练）的weights来初始化CNNs网络
        在NYUDv2训练集上用Couprie et.al.[1]定义的13类finetuned网络。用标准的随机梯度下降来优化，learning rate=0.01，momentum=0.9，weight decay=0.0005，在10k次迭代之后learning rate下降到0.001
        训练参数：mini-batch=64，迭代次数20K，GPU=Nvidia GTX Titan X.耗时2天。【注：为啥这么久？】
        
    B. Reconstruction Dataset
        文章自制了一个 small experimental RGB-D reconstruction dataset，用于重建一个办公室场景，数据相比NYUDv2更普遍，更具代表性，而不是只有一个简单的背景。
        文章开发了一个标注工具来标注13类数据。
        每100帧提取一帧作为验证测试帧。一共49个验证测试帧。
        文章给出两个test result table，一个是基于他们自制的reconstruction Dataset，一个是基于NYUDv2.
        
    C. CNN and CRF Update Frequency Experiments
        以RGB-CNN为测试网络
        在dataset上每2^n帧做预测，检验系统的run-time和准确率的关系。在每帧都做的时候（n=0），平均准确率由52.5%，最高，帧率8.2Hz，在128帧做一次检测时，平均准确率在45%+左右，帧率可达到30Hz+。
        Frames skipped by CRF = 500 时候对平均准确率有些许的改善，最高。太大会下降。【注：CRF是啥？】
    
    D. Accuracy Evaluation
        加入3D Slam之后，pixel-wise的prediction准确率比只做RGB单帧prediction的准确率要高4-5个点
    


---

## 2. 引用相关

### **2.1 Learning Deconvolution Network for Semantic Segmentation**[[Project page]](http://cvlab.postech.ac.kr/research/deconvnet/)[[Github]](https://github.com/HyeonwooNoh/DeconvNet)

```
@inproceedings{noh2015learning,
  title={Learning Deconvolution Network for Semantic Segmentation},
  author={Noh, Hyeonwoo and Hong, Seunghoon and Han, Bohyung},
  booktitle={Computer Vision (ICCV), 2015 IEEE International Conference on},
  year={2015}
} 
```

![屏幕快照 2016-11-11 下午10.06.37.png-282.3kB][2]


### **2.2 Predicting Depth, Surface Normals and Semantic Labels with a Common Multi-Scale Convolutional Architecture**[[Project page]](http://www.cs.nyu.edu/~deigen/dnl/)

```
David Eigen,      Rob Fergus
{deigen,fergus}@cs.nyu.edu
Paper PDF (ICCV 2015)
```

![屏幕快照 2016-11-11 下午10.11.07.png-601.1kB][3]


### **2.3 Indoor Semantic Segmentation using depth information**[[Author page]](http://perso.esiee.fr/~coupriec/publications.html)[[PDF]](https://arxiv.org/pdf/1301.3572v2.pdf)

```
C. Couprie, C. Farabet, L. Najman, and Y. LeCun, “Indoor semantic segmentation using depth information,” in Proceedings of the Interna- tional Conference on Learning Representations (ICLR), 2013
```
![屏幕快照 2016-11-11 下午10.26.53.png-627.5kB][4]

- 同一作者2014年发的一个journal：
```
Camille Couprie, Clement Farabet, Laurent Najman, Yann LeCun 
"Convolutional Nets and Watershed Cuts for Real-Time Semantic Labeling of RGBD Videos" 
Journal of Machine Learning Research, vol. 15, pp 3489-3511 (2014). PDF
```

### **2.4 Dense 3D Semantic Mapping of Indoor Scenes from RGB-D Images**[[Project Page]](http://www.vision.rwth-aachen.de/publication/0020/)

![Screenshot from 2016-11-14 12:20:40.png-126.3kB][5]

```
2014 ICRA Best Vision Paper
Alexander Hermans, Georgios Floros, Bastian Leibe
"Dense 3D Semantic Mapping of Indoor Scenes from RGB-D Images"
```



---

## 其他资料

[[ElasticFusion 论文解析]](http://www.2cto.com/kf/201605/509969.html)

[[Robot Learning Lab -- Cornell]](http://pr.cs.cornell.edu/sceneunderstanding/)
![Screenshot from 2016-11-14 12:28:22.png-242.3kB][6]
![Screenshot from 2016-11-14 12:29:03.png-71.6kB][7]


  [1]: http://static.zybuluo.com/lrl940607/7t0trfuew4vpuni29gh4dmim/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%20%E4%B8%8B%E5%8D%889.42.28.png

  [3]: http://static.zybuluo.com/lrl940607/xsnzdzic7rbt0srfvjeo9p6s/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%20%E4%B8%8B%E5%8D%8810.11.07.png
  [4]: http://static.zybuluo.com/lrl940607/lsp6nidtshaxmlyrmw5f2kbu/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%20%E4%B8%8B%E5%8D%8810.26.53.png
  [5]: http://static.zybuluo.com/lrl940607/wucphfx2o7dlq5u4e6lrvrp4/Screenshot%20from%202016-11-14%2012:20:40.png
  [6]: http://static.zybuluo.com/lrl940607/cyz0r0ck1msalr38t8vjque7/Screenshot%20from%202016-11-14%2012:28:22.png
  [7]: http://static.zybuluo.com/lrl940607/6uf3bctpn9m18hgvwr1o3mnl/Screenshot%20from%202016-11-14%2012:29:03.png