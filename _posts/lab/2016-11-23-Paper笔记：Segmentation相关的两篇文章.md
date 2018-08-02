---
layout: post
title: Paper笔记：Segmentation相关的两篇文章
category: lab
keywords: segmentation caffe
---

## 0. Predicting Depth, Surface Normals and Semantic Labels with a Common Multi-Scale Convolutional Architecture (dnl)
[[Code]](http://www.cs.nyu.edu/~deigen/dnl/)

    David Eigen      Rob Fergus
    {deigen,fergus}@cs.nyu.edu
    Paper PDF (ICCV 2015)

![Screenshot from 2016-11-23 14:46:45.png-208.4kB][1]

- 配置有点麻烦，装了好多东西，bug不断，不过error网上都可以解决。
- Requirements
    * theano
    * numpy, scipy
    * PIL or Pillow
- 先预测Depth和Normals，然后【image，Normals，Depth】一起预测Labels。每帧的处理时间大概20s/step。
- 文章主要是focus在normals，所以label的结果看上去并不怎么样。
- 两个step的网络输入都是320*240，输出是147*109,所以就会产生一个问题，就是第一步得到的Depth和Normals要自行resize后才可以扔到第二个网络里。demo code中没有给这一步，而是自带了一个文件包作为第二个网络的输入。
- 还有很奇怪的一点是，在预测label的demo code中发现自带的输入文件中Normals这个图片是有一圈【127，127，127】的灰度像素的，像素的宽度上下左右都不对称，这点不能理解/。
- 以及，用lab的数据测试后这个文章的自带model对于分割的效果并不好。

上结果图：
自带的demo图，label分40类：
![demo][2]
lab中的0.png测试图。label分40类：
![demolab40][3]
lab中的0.png测试图。label分13类：
![demolab13][4]

其中，文章给了3个model，分别对应以下三篇paper中定义的室内场景的4类，13类，以及40类：
4类：[[code&&data]](http://cs.nyu.edu/~silberman/projects/indoor_scene_seg_sup.html)

    "Ground in pink, Furniture in Purple, Props in Blue, Structure in Yellow"
    Indoor Segmentation and Support Inference from RGBD Images
    ECCV 2012
    
13类：

    Indoor Semantic Segmentation using depth information
    CVPR 2013
    
![Screenshot from 2016-11-23 20:10:03.png-14.5kB][5]

40类：
    
    NYUD-v2

## 1. Instance-aware Semantic Segmentation via Multi-task Network Cascades(MNC)
[[Github]](https://github.com/daijifeng001/MNC)

    @inproceedings{dai2016instance,
        title={Instance-aware Semantic Segmentation via Multi-task Network Cascades},
        author={Dai, Jifeng and He, Kaiming and Sun, Jian},
        booktitle={CVPR},
        year={2016}
    }
    
- won the first place in COCO segmentation challenge 2015, and test at a fraction of a second per image
![Screenshot from 2016-11-22 20:13:28.png-392.5kB][6]
![Screenshot from 2016-11-22 20:14:15.png-15.4kB][7]



## 2. Learning Rich Features from RGB-D Images for Object Detection and Segmentation(rcnn-depth)
[[Github]](https://github.com/s-gupta/rcnn-depth)

    @incollection{guptaECCV14,
      author = {Saurabh Gupta and Ross Girshick and Pablo Arbelaez and Jitendra Malik},
      title = {Learning Rich Features from {RGB-D} Images for Object Detection and Segmentation},
      booktitle = ECCV,
      year = {2014},
    }

- HHA的出处

## 3. detection/region/object proposal 方法综述文章 【2015】
http://blog.csdn.net/zxdxyz/article/details/46119369
![Screenshot from 2016-11-22 20:18:17.png-106.1kB][8]
以上都是可以找到源码的方法。
数据集方面，作者在PASCAL VOC07和ImagNet Detection dataset上面做了测试。
这里又有不少图，这里只贴一张AP的，其他请参考原论文咯。
![AP][9]

## 4. 数据集

### 4.1 ImageNet


  [1]: http://static.zybuluo.com/lrl940607/pqyzkh7rz02ewolckn76to7k/Screenshot%20from%202016-11-23%2014:46:45.png
  [2]: http://static.zybuluo.com/lrl940607/d10pl3nqvr8rpc8bsllapfa0/Screenshot%20from%202016-11-23%2019:55:14.png
  [3]: http://static.zybuluo.com/lrl940607/hiiinzivek5etmui2gmm7cn0/Screenshot%20from%202016-11-23%2019:59:38.png
  [4]: http://static.zybuluo.com/lrl940607/1txa5ihrcnq4cgnmk3g0rimv/Screenshot%20from%202016-11-23%2020:00:28.png
  [5]: http://static.zybuluo.com/lrl940607/h8yncq5za36vlf10nlppvi3z/Screenshot%20from%202016-11-23%2020:10:03.png
  [6]: http://static.zybuluo.com/lrl940607/7ca42ga3o8wu1kfb5ondiw40/Screenshot%20from%202016-11-22%2020:13:28.png
  [7]: http://static.zybuluo.com/lrl940607/jhxjiacbec36zv59f785uisa/Screenshot%20from%202016-11-22%2020:14:15.png
  [8]: http://static.zybuluo.com/lrl940607/jw6y74ytx2lqaskgp7kjro2z/Screenshot%20from%202016-11-22%2020:18:17.png
  [9]: http://static.zybuluo.com/lrl940607/bwegebd819rwrlqsw04yd7m8/Screenshot%20from%202016-11-22%2020:20:24.png