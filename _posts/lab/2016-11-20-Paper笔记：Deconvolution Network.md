---
layout: post
title: Paper笔记：Deconvolution Network
category: lab
keywords: segmentation caffe
---

## 文章基本信息：Learning Deconvolution Network for Semantic Segmentation
[[Project page]](http://cvlab.postech.ac.kr/research/deconvnet/)[[Github]](https://github.com/HyeonwooNoh/DeconvNet)
```
@inproceedings{noh2015learning,
  title={Learning Deconvolution Network for Semantic Segmentation},
  author={Noh, Hyeonwoo and Hong, Seunghoon and Han, Bohyung},
  booktitle={Computer Vision (ICCV), 2015 IEEE International Conference on},
  year={2015}
} 
```
![network structure][1]
![evaluation][2]

## 测试记录

1. 自带改写版caffe，里面写入了Deconvolution的相关结构
2. 基于VGG16网络，结构较大，需TITAN显卡的显存量
3. 主要代码目录结构
    - DeconvNet
        - inference
            - cache_FCN8s_results.m
            - generate_EDeconvNet_CRF_results.m
            - run_demo.m
4. 主要代码介绍
*run_demo.m*:
```matlab
clear all; close all; clc;

%% startup
startup;
config.imageset = 'test';
config.cmap= './voc_gt_cmap.mat';
config.gpuNum = 0;
config.Path.CNN.caffe_root = './caffe';
config.save_root = './results';

%% cache FCN-8s results
config.write_file = 1;
config.Path.CNN.script_path = './FCN';
config.Path.CNN.model_data = [config.Path.CNN.script_path '/fcn-8s-pascal.caffemodel'];
config.Path.CNN.model_proto = [config.Path.CNN.script_path '/fcn-8s-pascal-deploy.prototxt'];
config.im_sz = 500;

cache_FCN8s_results(config);

%% generate EDeconvNet+CRF results

config.write_file = 1;
config.edgebox_cache_dir = './data/edgebox_cached/VOC2012_TEST';
config.Path.CNN.script_path = './DeconvNet';
config.Path.CNN.model_data = [config.Path.CNN.script_path '/DeconvNet_trainval_inference.caffemodel'];
config.Path.CNN.model_proto = [config.Path.CNN.script_path '/DeconvNet_inference_deploy.prototxt'];
config.max_proposal_num = 50;
config.im_sz = 224;
config.fcn_score_dir = './results/FCN8s';

generate_EDeconvNet_CRF_results(config);
```
首先用FCN8s来做预测。输入是RGB图像，最长边要resize到500才能输入。输出有两个，一个是预测图results/FCN8s/results/\*.png，为展示效果。一个是results/FCN8s/score/\*.mat，为预测voc 21类的pixel-wise的score。预测图就是逐pixel取max score的类别生成的。这一步大概0.7s/帧

然后进行EDeconvNet+CRF的预测。在这之前有一步预处理，选出一些被选可能的bbox。类似selective search预先选一些proposal bbox 出来的方法。文章使用的是ECCV 2014的《Edge Boxes: Locating Object Proposals from Edges》方法[[Github]](https://github.com/pdollar/edges)[[一篇中文blog]](http://blog.csdn.net/wsj998689aa/article/details/39476551)。这个edge-boxes对一张图可以给出千量级的预选bbox，以及可信度及排序。文章取前50个bbox以控制预测时间。这一步很慢，大概5-20s/帧

以下是edge-boxes简介图，文章从edge的稀疏程度这个角度来生成box，具体请看以上的blog链接：
![Screenshot from 2016-11-20 15:29:59.png-889.8kB][3]

## 结果展示：

see : ~/DeconvNet/lab_result/results/results_fcn8s.mov


  [1]: http://static.zybuluo.com/lrl940607/1cyze4ay59ztwhkvohnpld0g/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%20%E4%B8%8B%E5%8D%8810.06.37.png
  [2]: http://static.zybuluo.com/lrl940607/y31uhvuowec54wneix42q06a/Screenshot%20from%202016-11-20%2015:04:50.png
  [3]: http://static.zybuluo.com/lrl940607/s4rf4bo1py6u8432c50xt88d/Screenshot%20from%202016-11-20%2015:29:59.png