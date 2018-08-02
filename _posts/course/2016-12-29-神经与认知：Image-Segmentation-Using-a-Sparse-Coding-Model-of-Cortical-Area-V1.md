---
layout: post
title: 神经与认知：Image Segmentation Using a Sparse Coding Model of Cortical Area V1
category: course
keywords: 神经与认知计算
---

http://tieba.baidu.com/p/2201184962
有一个现象叫endstopping：当一条线延伸出视觉皮质细胞感受野的时候，细胞的激活不增强，反而减弱。也就是说附近的同类细胞激活会抑制它。这个现象including V1 (area 17; refs 2, 3), V2
(area 18; refs 1,4), V4 (ref. 5) and MT6.都能发现
为什么有这种现象？
这里的基本假设是，皮层会通过删除可预测的数据来消除信息冗余。
因为自然图像的某个像素点和附近的信息总是相关的，某点的信息总是可以通过周围点来预测，比如周围点灰度的线性加权求和。这也解释了center–surround receptive fields in the retina and LGN ( LGN和视网膜的边缘检测算法)
同理，视觉信息在时间上也是相关的。可以通过过去时间点上数值的加权线性求和进行预测。颜色之间的预测也是如此。
用预测编码的分层模型Usinga hierarchical model of predictive coding, 显示出，这种“超感受野现象”可以用残差检测器来解释，反映了输入与预测的残差。

皮层上的所有神经元都可以看作是残差检测器，通过比较反馈预测与输入信号（求和？）决定自身的激活量，前馈信号是一个残差。
后一层的Predictive estimator通过残差修正预测
以次类推形成层级结构

---

RF (feedforward
classical receptive field (cRF)
predictive coding/(PC/BC)
lateral geniculate nucleus (LGN)
 
Here, a sparse coding algorithm, is applied to the task of perceptually salient boundary detection.

while derivative of Gaussian filters and Gabor filters have similar shapes to the RFs of orientation-tuned cells in primary visual cortex (V1) 

The positive and rectified negative responses are separated into two images X ON and X OF F simulating the outputs of cells in retina and LGN with circular-symmetric on-centre/off-surround and off-centre/on-surround RFs respectively.



## motivation 各种滤波都会产生冗余的边缘检测结果
    When a filter is applied to an image, the output shows a response at every location where there is a partial match, or partial overlap, between the filter and an image feature that the filter “detects”. This results in a dense representation, in which the same image feature is represented by multiple, redundant, filter responses. For example, consider using a Gabor filter bank to locate edges in a simple, artificial, image which contains a single non-zero pixel, as shown in the first column of the top-row of Fig. 1 1 . The maximum response across all filters at each pixel location is shown in the second column of Fig. 1. This is just the point-spread function of the filters. Active filters have a range of orientation preferences, centred at a range of different locations around the location of the non-zero pixel in the original image. The full range of responses can be visualised by using the filter outputs to reconstruct the edges they represent, i.e., the strength of each filter response at each location is used to determine the strength with which a short oriented line segment is added to the reconstructed image. This is shown in the third column of Fig. 1. Similar results for a range of different, artificial, images and for a patch taken from a natural image are shown in rows 2 to 5 of Fig. 1. In each case linear filtering generates a large range of redundant responses.

---

## connection between 神经边缘检测 && 图像滤波边缘检测技术：都是抑制那些光滑边缘旁边的点，以及增强需要的弱边（以canny举例说明两者的本质是相似的）

A neuron in such a model has a classical receptive field (cRF), often defined using a Gabor
function, that receives input from the image. The weights of the cRF are multiplied by the corresponding pixel
values such that the initial response of the neuron is identical to the output of a linear filter. However, this response
is subsequently modified by inputs to the non-classical receptive field (ncRF) which originate from neighbour-
ing neurons. Typically ncRF connections are excitatory from neurons in the local neighbourhood which have a
location and an orientation preference that is consistent with a smooth contour linking the two neuron’s cRFs,
and ncRF connections are inhibitory from neurons in the local neighbourhood which are not consistent with such
smooth contours. The inhibitory and excitatory ncRF connections thus perform operations analogous to the two
types of post-processing typically applied to linear filtering methods (as discussed above): suppressing unwanted,
redundant, responses, while enhancing weak, but wanted, responses by interpolating between edge elements to find extended contours. 

Current biologically-inspired edge-detection methods are therefore not radically different from traditional image processing techniques which employ linear filtering followed by post-processing operations.

## 很多方法不是摒弃掉了后处理（like canny）

These post-processing methods (whether biologically-inspired or not) are only partially successful in suppress-
ing unwanted filter responses while retaining filter responses that correspond to true boundaries. Hence, rather
than attempting to tidy up the output of a linear filtering process, many recent boundary-based image segmen-
tation algorithms have abandoned such filtering entirely

---

The proposed algorithm is an extension to the /biased competition (PC/BC) model of V1.

---

The success of the PC/BC model
of V1 in explaining cortical function is due to the responses of neurons not being independent of the responses of
other neurons. Instead, these neurons interact in a mutually suppressive and nonlinear manner to determine the
most efficient, sparse, representation of the input image (see Section 2). The method used to determine the sparse
subset of neurons that best explain the underlying causes of the sensory input is a form of perceptual inference
called predictive coding.

---

## 横向链接

Recurrent connections are used to take the outputs of the prediction neurons and use them as additional inputs to the V1 model

---

##Suppressing Texture：抑制纹理