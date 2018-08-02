---
layout: post
title: 记录：使用py-faster-rcnn在自己的数据集上 
category: lab
keywords: rcnn caffe segmentation detection
---

## **1. 首先介绍项目背景**

2016.9月开始加入一个腾讯项目--室内场景重建与分割，由@杨晟学长牵头。目标是将室内（比如办公室）的kincet扫描数据(RGB-D)作为输入，用点云重建出3D场景，然后分割出其中的重要组成部分--比如墙面、地面、桌子、椅子、以及桌子上面的一些零碎物件等

我承担的部分是在图像上做分割，**最初的愿景是用当下很火的神经网络技术，在RGB-D的输入下做分割**。在这个愿景下，我做了以下的尝试：

### **Fully Convolutional Networks for Semantic Segmentation(FCN) @CVPR 2016**[[Github链接]](https://github.com/shelhamer/fcn.berkeleyvision.org)

先上图看效果，以下均是用@杨晟的实拍数据，paper中没有自带RGB-D的demo测试图片。

![0.png-388.9kB][1]
![0.png-261.5kB][2]
![file:///home/dalong/smallRoom3/seg_result/0.png][3]

图1为RGB图，图2为kinect扫得的深度图，图3为分割结果。

- 图2的深度图是使用一种HHA的算法[[Github链接]](https://github.com/s-gupta/rcnn-depth/blob/master/rcnn/saveHHA.m)得到的.HHA的输入是kinect的深度图，输出就是以上的特征图(see HHA features from Gupta et al. ECCV14.)

- 这份工作使用的数据集为nyudv2数据集**（这是一个室内场景RGB-D的数据集，数据集提供了1400+的像素级的标注图片，以及大量的未标注图片）**，文章使用的数据集不是全部的nyuvdv2标注类别，而是其中的40类,所以要download文章所用的数据集如下。

> While there are many labels, we follow the 40 class task defined by
Perceptual Organization and Recognition of Indoor Scenes from RGB-D Images.
Saurabh Gupta, Pablo Arbelaez, and Jitendra Malik.
CVPR 2013
at http://www.cs.berkeley.edu/~sgupta/pdf/GuptaArbelaezMalikCVPR13.pdf .
To reproduce the results of our paper, you must make use of the data from Gupta et al. at http://people.eecs.berkeley.edu/~sgupta/cvpr13/data.tgz .

- 截至2016.11.7,文章的github仍然在更新，虽然有release的版本，但文件目录结构整理不完善，需要一些debug才能跑通。


**Instance-aware Semantic Segmentation via Multi-task Network Cascades(MNC) @CVPR 2016**[[Github链接]](https://github.com/Oh233)

待完善

## **2. 新数据集结构**
- ~/DATA_indoor/
    - coco/
        - ...
    - VOC2007_2012/
        - ImageList_train.txt
        - ImageList_test.txt
        - bootle/
        - ...
        - chair/
            - bbox/
                - 2007_000170.txt
                - ...
            - images/
                - 2007_000170.jpg
                - ...
            - selective_search/
                - 2007_000170.mat
                - ...

`ImageList_train.txt` looks like this:
> 2008_006136 bottle
> 2008_002864 bottle
> ...
> 2007_000170 chair
> ...

`2007_000170.txt` looks like this(first line is num of bbox, other lines are x1,y1,x2,y2):
> 4
> 360 192 381 265 
> 399 181 422 235 
> 270 180 291 247 
> 294 176 312 241 

what's in `2007_000170.mat` has the same format with the origin selective_search mat in py-faster-rcnn.

## **3. faster-rcnn 代码组织结构介绍**

### 1. 入口
 /home/dalong/py-faster-rcnn/experiments/scripts/faster_rcnn_end2end.sh
入口用法： 
```bash
$ ./experiments/scripts/faster_rcnn_end2end.sh 0 VGG16 pascal_voc
```
这个sh里面有
1.路径及迭代次数的设置
```bash
case $DATASET in
  pascal_voc) #之后这里需要改动
    TRAIN_IMDB="voc_2007_trainval"
    TEST_IMDB="voc_2007_test"
    PT_DIR="pascal_voc"
    ITERS=40000
    ;;
    ...
```
2.train的代码入口
```bash
$ time ./tools/train_net.py --gpu ${GPU_ID} \
  --solver models/${PT_DIR}/${NET}/faster_rcnn_end2end/solver.prototxt \
  --weights data/imagenet_models/${NET}.v2.caffemodel \
  --imdb ${TRAIN_IMDB} \
  --iters ${ITERS} \
  --cfg experiments/cfgs/faster_rcnn_end2end.yml \
  ${EXTRA_ARGS}
```
3.test的代码入口
```bash
$ time ./tools/test_net.py --gpu ${GPU_ID} \
   --def models/${PT_DIR}/${NET}/faster_rcnn_end2end/test.prototxt \
   --net ${NET_FINAL} \ #之后这里需要改动
   --imdb ${TEST_IMDB} \
   --cfg experiments/cfgs/faster_rcnn_end2end.yml \
   ${EXTRA_ARGS}
```
### 2. train_net.py
/home/dalong/py-faster-rcnn/tools/train_net.py
从入口开始追踪代码，就可以到这个文件了。这里面除了主函数有两个函数`def parse_args()`和`def combined_roidb(imdb_names)`，第一个函数使用来加载参数的，第二个函数是用来读取数据的。
```python
def combined_roidb(imdb_names):
    def get_roidb(imdb_name):
        imdb = get_imdb(imdb_name) # 这里是加载data的入口
        print 'Loaded dataset `{:s}` for training'.format(imdb.name)
        imdb.set_proposal_method(cfg.TRAIN.PROPOSAL_METHOD)
        print 'Set proposal method: {:s}'.format(cfg.TRAIN.PROPOSAL_METHOD)
        roidb = get_training_roidb(imdb)
        return roidb

    roidbs = [get_roidb(s) for s in imdb_names.split('+')]
    roidb = roidbs[0]
    if len(roidbs) > 1:
        for r in roidbs[1:]:
            roidb.extend(r)
        imdb = datasets.imdb.imdb(imdb_names)
    else:
        imdb = get_imdb(imdb_names)
    return imdb, roidb
```
从`train_net.py`的文件头找到`get_imdb`的来源
```python
from datasets.factory import get_imdb
```
### 3. factory.py
/home/dalong/py-faster-rcnn/lib/datasets/factory.py
这个文件的全部代码贴在下面：
```python
"""Factory method for easily getting imdbs by name."""

__sets = {}

from datasets.pascal_voc import pascal_voc #这里找到默认pascal_voc数据集的处理文件
from datasets.coco import coco
import numpy as np

# Set up voc_<year>_<split> using selective search "fast" mode
for year in ['2007', '2012']:
    for split in ['train', 'val', 'trainval', 'test']:
        name = 'voc_{}_{}'.format(year, split)
        __sets[name] = (lambda split=split, year=year: pascal_voc(split, year)) #从get_imdb函数走到了这里，所以pascal_voc这个类才是重点！这里的split参数在训练模式和测试模式下分别为'trainval'和'test'

# Set up coco_2014_<split>
for year in ['2014']:
    for split in ['train', 'val', 'minival', 'valminusminival']:
        name = 'coco_{}_{}'.format(year, split)
        __sets[name] = (lambda split=split, year=year: coco(split, year))

# Set up coco_2015_<split>
for year in ['2015']:
    for split in ['test', 'test-dev']:
        name = 'coco_{}_{}'.format(year, split)
        __sets[name] = (lambda split=split, year=year: coco(split, year))

def get_imdb(name): #这里传进来的name参数是faster_rcnn_end2end.sh中的TRAIN_IMDB="voc_2007_trainval"
    """Get an imdb (image database) by name."""
    if not __sets.has_key(name):
        raise KeyError('Unknown dataset: {}'.format(name))
    return __sets[name]()

def list_imdbs():
    """List all registered imdbs."""
    return __sets.keys()
```

### 4. pascal_voc.py
/home/dalong/py-faster-rcnn/lib/datasets/pascal_voc.py
最终的关注点落在了这个类上。以下记录改动。

注：如果仅仅想在voc的21类上面取部分的几类进行训练而无其他改动的话，只需要改这个文件中以下两处：
1. In 'def __init__(self, image_set, year, devkit_path=None)'
```python
self._classes = ('__background__', # always index 0
                         'bottle','diningtable','pottedplant','sofa','chair','tvmonitor')
```
2. In 'def _load_pascal_annotation(self, index)'
```python
### 以下两行是增加的
valid_class_objs = [obj for obj in objs if obj.find('name').text in self._classes]
objs = valid_class_objs
###

num_objs = len(objs)

boxes = np.zeros((num_objs, 4), dtype=np.uint16)
gt_classes = np.zeros((num_objs), dtype=np.int32)
overlaps = np.zeros((num_objs, self.num_classes), dtype=np.float32)
# "Seg" area for pascal is just the box area
seg_areas = np.zeros((num_objs), dtype=np.float32)

```

## **4. faster-rcnn train 部分代码改动**

### 1. pascal_voc.py -> indoor_data.py
重写这个类，这里仅跟自己的数据格式有关，很多函数都有路径上的改动。以及原代码是针对VOC中的21类，我的数据集里面是7类，所以对于类别不同的改动只在初始化函数里面做就好了。

这里不放代码了。。太长了。。改的地方很零散。。

### 2. factory.py
增加如下代码：
```python
from datasets.indoor_data import indoor_data

__sets['indoor_data_train'] = (lambda imageset = 'indoor_data_train': indoor_data(imageset))
__sets['indoor_data_test'] = (lambda imageset = 'indoor_data_test': indoor_data(imageset))
```

### 3. faster_rcnn_end2end.sh
在case中增加如下indoor_data代码块：
```bash
case $DATASET in
  pascal_voc)
    TRAIN_IMDB="voc_2007_trainval"
    TEST_IMDB="voc_2007_test"
    PT_DIR="pascal_voc"
    ITERS=40000
    ;;
  indoor_data) ## 新增代码块
    TRAIN_IMDB="indoor_data_train"
    TEST_IMDB="indoor_data_test"
    PT_DIR="pascal_voc"
    ITERS=40000
    ;; ##
```
之后就可以用以下命令调用代码进行训练了：（注：这仅仅是训练模型这一步！！！）
```bash
$ ./experiments/scripts/faster_rcnn_end2end.sh 0 VGG16 indoor_data
```

## **5. faster-rcnn test 部分代码改动**
以下是想要在自己的数据集上测试的改动。
### 关键源代码 voc_eval.py

In `pascal_voc.py`:

```python
def _do_python_eval(self, output_dir = 'output'):
    ...
    rec, prec, ap = voc_eval(
                filename, annopath, imagesetfile, cls, cachedir, ovthresh=0.5,
                use_07_metric=use_07_metric)
    ...
```
使用默认参数时： 
```python
filename = '$FASTER_RCNN_ROOT/data/VOCdevkit2007/results/VOC2007/Main/comp4_det_test_bootle.txt'
annopath = '$FASTER_RCNN_ROOT/data/VOCdevkit2007/VOC2007/Annotations/{:s}.xml'
imagesetfile = '$FASTER_RCNN_ROOT/data/VOCdevkit2007/VOC2007/ImageSets/Main/test.txt'
cls = 'bottle'  # (or 'car'/'person'/'chair'...)
cachedir = '$FASTER_RCNN_ROOT/data/VOCdevkit2007/annotations_cache'
ovthresh = 0.5
use_07_metric = True
```
我们需要进入`voc_eval.py`文件中修改`voc_eval`中的部分
所以这里我我们新建了一个`indoor_eval.py`的文件，把以上入口处改成`voc_eval`即可。

详细改动同样不放在这里了。零散的路径改动。。。





  [1]: http://static.zybuluo.com/lrl940607/d5cy58os8dgtirsjptt4mv9i/0.png
  [2]: http://static.zybuluo.com/lrl940607/onzrx9lwj4qdz2tiddx7hzys/0.png
  [3]: http://static.zybuluo.com/lrl940607/gydmcahn66gb5b7k5tjkn77k/0.png