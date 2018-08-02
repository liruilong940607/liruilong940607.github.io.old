# [For Zixi] 寒假Task2018

标签（空格分隔）： 2018寒假

---

### 1. 复现MaskRcnn开源代码

千呼万唤始出来，四天前FaceBook终于开源了MaskRcnn[\[Github\]][1], 是基于Caffe2的。

我们第一件事是用他开源的代码复现他的论文[\[ArXiv\]][2]中的结果。就是用他的代码train一遍。看看能不能复现论文中的分数。不能的话，差多少，原因是什么。关注github上的issue，因为应该会有很多人会尝试复现这个。

复现之后，要做几个对比实验。这里我先写一个对比实验吧，开头比较难搞一点。

### 2. 对比实验

把他的RoiAlignLayer换成我们的AffineAlignLayer。其他不动，包括预测bbox的分支也保留。train一下看看能到多少分。就是原本从特征图上抠出一块的操作，变成用关键点对齐的操作。并且concat上关键点的feature

注意：原来的RoiAlign是train的时候用了gt+predict的bbox。我们的AffineAlign在train的时候用gt+predict（另一个模型predict好的）的关键点

这个对比实验有点麻烦因为他的code是caffe2的。我的AffineLayer是pytorch的。你可以基于caffe2，pytorch或者你熟悉的tf做这个实验都ok。当然，对你本身最好的方式，是挑一个你熟悉的框架复现一遍maskrcnn，复现的过程中遇到问题就是学习的过程。毕竟作者已经放了参考答案，所以应该没什么克服不了的问题。

当然这个实验希望证明的是Align时，用关键点对齐比用bbox对齐能够给出更好的分割结果。如果实验结果不理想，AlignLayer还可以针对地做点优化，这个做完实验再看。



[1]: https://github.com/facebookresearch/Detectron
[2]: https://arxiv.org/pdf/1703.06870.pdf


