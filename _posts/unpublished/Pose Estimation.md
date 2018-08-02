# Pose Estimation

标签（空格分隔）： 未分类

---

已有的工作主要有三种思路：

- openpose：CNN卷出全图所有的点heatmap + 所有连接heatmap（PAF）；然后通过后处理连出每一个人。
- two-stage：person detection + single person heatmap predict
- Mask-rcnn：也是person detection + single person heatmap predict，不过是将两者融在了一个网络里。一起输出。一起训练。

优缺点评价：

two-stage的方法：很依赖detection的结果。虽然detection误检测的人在第二个stage可以凭借pose检测器的响应筛掉一些，但是detection漏检测的人是找不回来的（主要是交叠的人，iou nms会直接抑制掉。由此也产生了oks-based nms）。凭借two-stage的方法一般很慢，图片要走两次feature extractor。刷榜前几名都是这种思路，因为two-stage比较好分工合作，可控，并且可以两个stage都用resnet做backbone，达到最大化模型的学习能力。

openpose：预测点+连接。这个思路之前也有过一些工作（所有点全连接）。这种方式的好处是避开了detection有人会被nms的问题，并且不需要像two-stage那样每个人过一遍pose检测器，使得效率变得与人数无关（可以做到10+FPS）。缺点是需要后处理来连接确定所有点的归属，当PAF响应较弱时会连错，当关键部位的点的响应较弱时，一个人会被连成多个部分。（断掉）

Mask-rcnn：基于faster-rcnn，加了一个branch使得detection可以和mask／keypoints一起做。coco2017三榜有名。优点是所有东西可以一起出来，并且轻微的改善了two-stage的完全依赖detection的问题。（keypoints和detection同时训练可以互相改善）不过主体上还是依赖detection。用resnet101-FPN做backbone可以做到3-5FPS。缺点是backbone和后面的branch挤在一个网络里，12G的单卡限制使得网络做不大，而且batchsize基本只能在1-2之间徘徊。再加上其中的rpn很难调，实测很难训很难训。。。。

我准备着手的点：

- Mask-rcnn + openpose: 做keypoints
自底向上和自顶向下的结合。两者都可以通过一个网络出multi person的结果。优缺点可以互补（openpose的连接断掉的问题，在detection中不存在，detection中的人数个数问题，框的漂移导致手被砍掉，以及nms的问题在openpose中不存在）。 
网络结构大概有几种设计思路，还需要coding和实验。

- mask+keypoints：基于openpose做instance segmentation
这是个很新的思路，没有什么reference。之前的instance segmentation都是先detection，再segmentation（例Mask-rcnn）。不过由于人这一类比较特殊，于coco有keypoints的标注，所以我们可以利用keypoints来区分instance。绕过detection，做mask+keypoints的instance segmentation。这个思路还在考虑中。。









