---
layout: post
title: windows python & jupyter 安装简明教程
category: tutorial
keywords: windows
---

### 1. python 安装

（已完成，跳过）

### 2. setuptools 安装（pip安装的前提）
- 下载地址：https://pypi.python.org/packages/f4/21/9990132a250d49190c2fb41ea2adb9b4693c0b10ec1b92db476d3f06e6cb/setuptools-38.3.0.zip#md5=ea9a655779971d3904d88f067b7b830e

- 解压，用cmd控制台进入解压目录，输入
```
python setup.py install
```

### 3. pip 安装（python的包管理工具）

- 先确认是否已安装pip： 进入命令行cmd， 输入pip。如果得到以下提示，则需要安装pip。
![Screen Shot 2018-01-05 at 5.41.21 PM.png-67.7kB][1]

- 下载地址：https://pypi.python.org/packages/11/b6/abcb525026a4be042b486df43905d6893fb04f05aac21c32c638e939e447/pip-9.0.1.tar.gz#md5=35f01da33009719497f01a4ba69d63c9

- 下载完成之后，解压到一个文件夹，用cmd控制台进入解压目录，输入：
```
python setup.py install
```
- 安装完成后, 要在系统环境变量`Path`里添加以下路径。
```
C:\Python27\Scripts;
```
如图：
![image.png-253.3kB][2]

### 4. 安装依赖
- 安装VC 9.0（词云依赖）: 下载之后双击安装
下载地址：https://www.microsoft.com/en-us/download/confirmation.aspx?id=44266

- 安装作业1所需的python包
进入cmd, 进入`requirements.txt`文件所在目录
```
pip install -r requirements.txt
```

### 5. 完成。运行jupyter本地服务。
cmd进入作业1文件所在目录。输入：
```
jupyter notebook
```
弹出网页即成功。

### 6. 最后一件事。打开hw1.ipynb后，修改里面的字体路径为`C:\Windows\Fonts\SIMSUN.TTC`：
![image.png-210.7kB][3]




  [1]: http://static.zybuluo.com/lrl940607/bmh3x2e5ouhin9p6a71fgvfg/Screen%20Shot%202018-01-05%20at%205.41.21%20PM.png
  [2]: http://static.zybuluo.com/lrl940607/je55s7njrgyukcy1vgymfmb1/image.png
  [3]: http://static.zybuluo.com/lrl940607/vyh3d5lbt91gjj52fxookb5v/image.png