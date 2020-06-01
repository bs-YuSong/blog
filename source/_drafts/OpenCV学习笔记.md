---
title: OpenCV学习笔记(1)
tags: 
  - Python
  - 笔记
categories: OpenCV
---

### 什么是图片？

1. 图像的基本数据
a> 通道与数目：
b> 高与宽
c> 图像的类型
d> 像素数据
````python
def getImageInfo(img):
	print(type(img))
	print(img.shape)
	print(img.size)
	print(img.dtype)
	height = mg.shape[0]
	weight = img.shape[1]
	channels = img.shape[2]
	print("height: %s,weight: %s,channels: %s"%(height, weight, channels))
````

2. 安装opencv-contrib-python
````
pip install opencv-contrib-python
````

3. 导入，图片存读、转化灰度
````python
import cv2

img = cv2.imread(filePath)	// 读取
cv2.imshow(title, img)		// 展示
cv2.imwrite(filePath, img)	// 存储
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)	// 转化为灰度
````

