---
layout: post
title: Mac下opencv2+python2.7 & opencv3+python3.6的配置
category: tutorial
keywords: mac
---

## 1. Install ffmpeg

为了支持VideoWriter类，首先安装ffmpeg：
```
$ brew install ffmpeg
```
会自动安装一些ffmpeg的依赖。亲测这样安装的ffmpeg支持以下解码方式：
```
[opencv2] videoWriter = cv2.VideoWriter('output.avi', cv2.cv.CV_FOURCC('M', 'J', 'P', 'G'), fps, size)  
[opencv3] videoWriter = cv2.VideoWriter('output.avi', cv2.VideoWriter_fourcc('M', 'J', 'P', 'G'), fps, size)  
```
当avi -> mp4时，虽然也会正确写出一个mp4文件，但是会报红一条错。当使用源码编译时，红错消失。但是flv的input无法解析。所以最后还是用brew安装的ffmpeg。
附上ffmpeg源码编译的参考：http://trac.ffmpeg.org/wiki/CompilationGuide/macOS
```
$ brew install automake fdk-aac git lame libass libtool libvorbis libvpx opus sdl shtool texi2html theora wget x264 x265 xvid yasm

$ git clone http://source.ffmpeg.org/git/ffmpeg.git ffmpeg
$ cd ffmpeg
$ ./configure  --prefix=/usr/local --enable-gpl --enable-nonfree --enable-libass \
--enable-libfdk-aac --enable-libfreetype --enable-libmp3lame \
--enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libopus --enable-libxvid
$ make && sudo make install
```

## 2. 编译opencv 
python2.7 + opencv2.4.13: 

```
$ source ~/env-p2.7/bin/activate
$ cd ~/3rd-Party/opencv/opencv-2.4.13.3
$ mkdir release && cd release
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D PYTHON_PACKAGES_PATH=~/env-p2.7/lib/python2.7/site-packages \
-D PYTHON_LIBRARY=/usr/local/Cellar/python/2.7.13_1/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib \
-D CMAKE_INSTALL_PREFIX=../../opencv2-py2.7 ..
```
编译完看一下编译信息，ffmpeg和python部分都无误后，
```
$ make -j16
$ make install (UNDO)
```
由于只需要cv2.so，所以我没有install，如果install的话，会安到../../opencv2-py2.7目录下。
```
$ cd ~/env-p2.7/lib/python2.7/site-packages/
$ cp ~/3rd-Party/opencv/opencv-2.4.13.3/release-py2.7/lib/cv2.so ./cv2.opencv-2.4.13.3.so 
$ ln -s cv2.opencv-2.4.13.3.so cv2.so
```
至此大功告成。

**测试：** pytest.py
```
import cv2

input = '/Users/dalong/Downloads/1488511789.flv'
output = input+'.avi'
videoCapture = cv2.VideoCapture(input)  
fps = videoCapture.get(cv2.cv.CV_CAP_PROP_FPS)  
size = (int(videoCapture.get(cv2.cv.CV_CAP_PROP_FRAME_WIDTH)), int(videoCapture.get(cv2.cv.CV_CAP_PROP_FRAME_HEIGHT)))  
videoWriter = cv2.VideoWriter(output, cv2.cv.CV_FOURCC('M', 'J', 'P', 'G'), fps, size)  
success, frame = videoCapture.read()      
while success :  
    videoWriter.write(frame)
    success, frame = videoCapture.read()   

```

---

python3.6 + opencv3.3:

```
$ source ~/env-p3.6/bin/activate
$ cd ~/3rd-Party/opencv/opencv-3.3.0
$ mkdir release && cd release
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=../../opencv3-py3.6 \
-D PYTHON3_PACKAGES_PATH=~/env-p3.6/lib/python3.6/site-packages \
-D PYTHON3_LIBRARY=/usr/local/Cellar/python3/3.6.2/Frameworks/Python.framework/Versions/3.6/lib/libpython3.6m.dylib \
-D PYTHON3_INCLUDE_DIR=/usr/local/Cellar/python3/3.6.2/Frameworks/Python.framework/Versions/3.6/include/python3.6m \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D BUILD_EXAMPLES=ON \
-D BUILD_opencv_python3=ON \
-D PYTHON_DEFAULT_EXECUTABLE=~/env-p3.6/bin/python \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules ..
```
编译完看一下编译信息，ffmpeg和python部分都无误后，
```
$ make -j16
$ make install (UNDO)
```
由于只需要cv2.so，所以我没有install，如果install的话，会安到../../opencv3-py3.6目录下。
```
$ cd ~/env-p3.6/lib/python3.6/site-packages
$ cp ~/3rd-Party/opencv/opencv-3.3.0/release-py3.6/lib/python3/cv2.cpython-36m-darwin.so  ./
$ ln -s cv2.cpython-36m-darwin.so  cv2.so
```
至此大功告成。

**测试：** py3test.py
```
import cv2
input = '/Users/dalong/Downloads/1488511789.flv'
output = input+'.avi'
videoCapture = cv2.VideoCapture(input)  
fps = videoCapture.get(cv2.CAP_PROP_FPS)  
size = (int(videoCapture.get(cv2.CAP_PROP_FRAME_WIDTH)), int(videoCapture.get(cv2.CAP_PROP_FRAME_HEIGHT)))  
videoWriter = cv2.VideoWriter(output, cv2.VideoWriter_fourcc('M', 'J', 'P', 'G'), fps, size)  
success, frame = videoCapture.read()      
while success :  
    videoWriter.write(frame)
    success, frame = videoCapture.read()   

```

Ref: http://www.pyimagesearch.com/2015/06/29/install-opencv-3-0-and-python-3-4-on-osx/










