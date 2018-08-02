# A Review Of Some Landmark CNN Works

标签（空格分隔）： Machine-Learning

---
[TOC]

---
## 1. Gradient-Based Learning Applied to Document Recognition

## 2. Best Practices for Convolutional Neural Networks Applied to Visual document Analysis

This paper describes a set of concrete best practices that document analysis researchers can use to get good results with neural networks.

The most important practice is getting a training set as large as possible: adding distorted data(elastic distortions)

The next most important practice is that convolutional neural networks are better suited for visual document tasks than fully connected networks.

achieve the best performance on a handwriting recognition task (MNIST)

random distortion field and gauss blur (use sigma to control the field to be more smooth or more random)

It's good for MNIST because the images after distortion are meanful. They still looks like a realy hand-writting digital image. So this cause me to think: Is this method suitable for image segmentation job?(because segmentation is what I'm now focusing on) The answer to my mind is 'NO'. because distortion is not suitable for real-world images. It's suitable for human-create images, which have more varient struture! That's the point! 

