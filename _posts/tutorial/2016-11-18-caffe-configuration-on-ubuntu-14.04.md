---
layout: post
title: caffe ubuntu 14.04 -- 2016.11.18
category: tutorial
keywords: caffe
---

### 1. Install Dependencies

```bash
$ sudo apt-get install build-essential
$ sudo apt-get install cmake git vim

$ sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler 

$ sudo apt-get install python-dev python-pip
$ sudo apt-get install python-numpy

$ sudo apt-get install mesa-utils

```
### 2. Install Driver( cuda 7.5 and cuda 8.0 can install driver automaticlly, so skip this step)

easy way to install:
```bash 
$ sudo apt-get install nvidia-367 
```
查看显卡型号
```bash
$ lspci |grep VGA
```
官网下载驱动***.run
```bash
$ chmod +x ./***.run
$ ./***.run
```
安装提示关闭X server
```bash
$ sudo service lightdm stop
$ ## sudo service lightdm start(command for start)
```
再次安装提示Nouveau冲突，继续，NVIDIA会自动配置屏蔽Nouveau，然后需要重启电脑
```bash
$ sudo reboot
```
重新安装。有一些question如下描述，最后成果安装。
I did get the error about the pre-install script failing. I continued anyway. I got asked "Would you like to run the nvidia-xconfig utility to automatically update your X configuration file so that the NVIDIA X driver will be used when you restart X? Any pre-existing X configuration file will be backed up" I answered "Yes" and continued. I have a 64-bit system and got 32-bit errors. I didn't worry about it and continued. The 64-bit ones installed fine. I got to the end of the installation!

after install NVIDIA driver and **reboot**:
```bash
$ nvidia-smi
```
![Screenshot from 2016-11-18 14:03:44.png-35.4kB][1]

---
### 3. Install CUDA

官网下载cuda：cuda-repo-ubuntu1404-7-0-local_7.0-28_amd64.deb
```bash
$ sudo dpkg -i cuda-repo-ubuntu1404-7-0-local_7.0-28_amd64.deb
$ sudo apt-get update
$ sudo apt-get install cuda
```

安装完成后需要在/$HOME/.profile中添加环境变量 ## /etc/profile
```bash
$ sudo gedit /$HOME/.profile
```
在文件最后添加:
```vim
PATH=/usr/local/cuda/bin:$PATH
export PATH
```
保存后, 执行下列命令, 使环境变量立即生效

```bash
$ source /$HOME/.profile
```
同时需要添加lib库路径： 在 /etc/ld.so.conf.d/加入文件cuda.conf
```bash
$ sudo gedit /etc/ld.so.conf.d/cuda.conf
```
添加内容：
```vim
/usr/local/cuda/lib64
```
保存后，执行下列命令使之立刻生效：
```bash
$ sudo ldconfig
```

此处可以安装cuda-samples来测试cuda是否成功安装。

---
### 4. Install Atlas 

```bash
$ sudo apt-get install libatlas-base-dev
```
---
### 5. Install OpenCV 

解压到任意目录
```bash
$ unzip opencv-2.4.9.zip
```
进入源码目录，创建release目录
```bash
$ cd opencv-2.4.9
$ mkdir release  
```
可以看到在OpenCV目录下，有个CMakeLists.txt文件，需要事先安装一些软件
```bash
$ sudo apt-get install build-essential cmake libgtk2.0-dev pkg-config python-dev python-numpy libavcodec-dev libavformat-dev libswscale-dev  
```
进入release目录，安装OpenCV是所有的文件都会被放到这个release目录下
```bash
$ cd release
```
cmake编译OpenCV源码，安装所有的lib文件都会被安装到/usr/local目录下
```bash
$ cmake -D BUILD_TIFF=ON -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..  
```
安装
```bash
$ sudo make install  
```

Ref link: https://my.oschina.net/u/1757926/blog/293976

在GeForce GTX TITAN 上如上安装opencv时会报错：
Unsupported gpu architecture 'compute_11'  
解决方法：
When using cmake to do configurations, set the option CUDA_GENERATION to specific your GPU architecture. I ran across the same error and tried this to work out the problem.
this worked for me and shows a possible value for CUDA_GENERATION:
```bash
$ cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D CUDA_GENERATION=Kepler ..
```

安装2.4.9报错：
error: a storage class is not allowed in an explicit specialization
解决方法：
www.linuxidc.com/Linux/2015-04/116444.htm
但是其链接的文件失效了。其解释了这是2.4.9与cuda兼容的一个bug，所以之后改成安装2.4.13

---
### 6. Install Python && Download caffe

先安装相关依赖项
```bash
$ sudo apt-get install python-dev python-pip
```
Download caffe from Github:
```bash
$ cd ~
$ git clone https://github.com/BVLC/caffe.git
```
转到下载的caffe的目录下，然后转到python目录下
```bash
$ cd caffe
$ cd python
```
执行如下命令：
```bash
$ for req in $(cat requirements.txt); do sudo pip install $req; done
```

- ImportError: No module named packaging.version
    - sudo python -m pip install -U pip

---
### 7. Install Matlab : R2012b_UNIX-MacOS.iso

```bash
$ mkdir ~/iso
$ sudo mount -t iso9660 -o loop ~/Downloads/Matlab_R2012b_UNIX/R2012b_UNIX-MacOS.iso ~/iso/
$ cd ~/iso
$ sudo ./install
$ sudo umount ~/iso
```

注： 远程服务器安装时，要用ssh将X界面传输过来，所以要在ssh时加上-X（大写）参数

```bash
$ cd /usr/local/bin/
$ sudo ln -s ~/Matlab/R2012b/bin/matlab matlab
```
- Could not initialize shared resources for X11GraphicsDevice
    - Type opengl('save','software') at the MATLAB command

---
### 8. Install CUDNN 

if use cuda-8.0(nvidia 1080), must use cudnn-8.0
(https://developer.nvidia.com/rdp/cudnn-download)

  cudnn-7.0-linux-x64-v4.0-rc.tgz(use this)
  cudnn-6.5-linux-R1.tgz

下载后解压缩，转到该目录下，执行：
```bash
$ sudo cp lib* /usr/local/cuda/lib64/
$ sudo cp cudnn.h /usr/local/cuda/include/
```
更新软链接
```bash
$ cd /usr/local/cuda/lib64/
$ sudo rm -rf libcudnn.so libcudnn.so.4
$ sudo ln -s libcudnn.so.4.0.4 libcudnn.so.4
$ sudo ln -s libcudnn.so.4 libcudnn.so
```
注：以上4.0.4一类的版本号要更改

---
### 9. 编译caffe

```bash
$ cd caffe
$ make all -j8
$ make pycaffe
$ make matcaffe
$ make test -j8
$ make runtest
```

---

### 10. make matcaffe warning
由于安装的R2012b,适配的gcc版本低于系统自带gcc版本而报warning,用gcc-4.4替代系统的gcc-4.8：

Install GCC 4.4

First, install GCC 4.4 (and friends):
```bash
$ apt-get install gcc-4.4 g++-4.4 g++-4.4-multilib gcc-4.4-multilib
```
Set 4.4 to be the default

Then set 4.4 to be higher priority than 4.8:
```bash
$ # update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.4 100
$ # update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.6 50
$ update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 100
$ update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 50
$ # update-alternatives --install /usr/bin/cpp cpp-bin /usr/bin/cpp-4.4 100
$ # update-alternatives --install /usr/bin/cpp cpp-bin /usr/bin/cpp-4.6 50
```
Verify that it has worked:
```bash
$ gcc --version
```

### 11. matlab error:  matlab version `GLIBCXX_3.4.15' not found

solution:
http://wenku.baidu.com/link?url=wYcIQYWV4RIbOzU84cAMuM1bOPb7sTX5qxUL2SKvi0wYP3ddzTLE4B2ktrAtHK3Hhijzhz2GJPTKFfKj0gwxCu0mIzvXjzeiU3F4-mQ_3qm

### 12. nautilus /media/

以窗口形式打开某个文件夹

### 13 jupyter remote
 ref: http://blog.csdn.net/bitboy_star/article/details/51427306
 
### 14 任务：查看非root运行的进程

 ps -U root -u root -N

  [1]: http://static.zybuluo.com/lrl940607/p2poc81ohnumwvvzpiy3k5w8/Screenshot%20from%202016-11-18%2014:03:44.png