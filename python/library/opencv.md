---
title: opencv
date: 2016-03-29 20:01:38
tags: [python]
---

# 简述
因为比赛的需要，学习图像识别，就用比较简单的Python来编写
# 安装
python2.7 安装 opencv 参考：
<http://www.jianshu.com/p/67293b547261>

# 像素访问
opencv图像是二维或者三维numpy.array
8-bit 灰度图像是一个二维数组
24-bit BGR图像是一个三位数组
cv2中对图片像素的访问和在numpy中一样
```
img[<height>, <width>, <channel>]
```
三个参数分别表示行，列，通道

# 读取并显示图像
## 最简单的程序
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"显示一个图形"

import cv2

img = cv2.imread("pic.jpg")
cv2.namedWindow("Image")
cv2.imshow("Image", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
### imread
载入图片
```
imread(<filename>, <flags>)
```
以BGR(blue-green-red)格式读取图片，和RGB类似，不过顺序相反
第一个参数是文件名
第二个参数是载入标识
1 : cv2.IMREAD_COLOR (BGR) (默认) 
0 : cv2.IMREAD_GRAYSCALE (grayscale)
-1 : cv2.IMREAD_UNCHANGED (由图片本身决定)

+ flags >0返回一个3通道的彩色图像。
+ flags =0返回灰度图像。
+ flags <0返回包含Alpha通道的加载的图像。

### namedWindow
创建一个窗口
```
namedWindow(<winname>, <flags>)
```
第一个参数是窗口名称
第二个参数是窗口标识，通常使用默认
可以调用**destroyWindow()**或者**destroyAllWindow()**关闭窗口
对于已经存在的同名窗口，函数不做任何事

### imshow
在指定窗口显示一幅图像
```
imshow(<winname>, <inputarrat>)
```
第一个参数是窗口名称
第二个参数是显示的图像

### imwrite
输出图像到文件
```
imwrite(<filename>, <inputarray>, <params>)
```
第一个参数是文件名，带上后缀，比如“.jpg”
第二个参数是图像数据
第三个参数是参数编码，一般不需要填写

+ 对于JPEG格式图片，这个参数表示0到100的图片质量(IMWRITE_JPEG_QUALITY)**(1)**，默认95
+ 对于PNG格式图片，这个参数表示压缩级别(IMWRITE_PNG_COMPRESSION)**(16)**从0到9，默认3
+ 对于PPM，PGM，PBM格式的图片，这个参数表示一个二进制格式标志(IMWRITE_PXM_BINARY)**(32)**，取值0或1，默认为1

```
#cv2.imwrite("lalala.jpg", img, [1,20])
cv2.imwrite("lalala.jpg", img, [cv2.IMWRITE_JPEG_QUALITY,20])
```
## 图像操作
### img.shape
```
(<height>, <width>, <channel>)
```
返回值是一个元组，对应的值是图片的高，宽，通道
### img.ndim查看图片维度
```
#椒盐现象
def salt(img, n):
    for a in range(n):
        i = int(random.random() * img.shape[0])
        j = int(random.random() * img.shape[1])
        img[i , j] = [255, 255, 255] #blue, green, red
```
### img.split
分离通道
```
b, g, r = cv2.split(img)
```
或者
```
#这里是镜像
b = img[:, :, 0]
g = img[:, :, 1]
r = img[:, :, 2]
```
### cv2.merge
通道合并
```
img = cv2.merge([b, g, r])
```
### np.zeros
opencv中没有createimage接口，创建图像需要调用numpy的函数
```
img = np.zeros(<img.shape>, np.uint8)
```
例如创建一个高500，宽400，3通道的图片
```
img = np.zeros((500, 400, 3), np.uint8)
```
### img.copy()
通过拷贝一个numpy的array，拷贝一个图片

### cv2.cvtColor
改变颜色编码(color-space)
```
cv2.cvtColor(<image>, <flag>)
```
BGR -> GRAY
```
img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```
BGR -> HSV
```
img = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
```
### cv2.calcHist
绘制直方图
```
cv2.calcHist([image], channels, mask, histSize, ranges)
```

+ image: 输入图像，用方括号括起来
+ channels: 通道数
+ mask: 掩膜图像，如果是整幅图，为None
+ histSize: 灰度级个数
+ ranges: 直方图像素值范围，通常为[0, 256]

## 图像处理
### cv2.erode
```
img = cv2.imread("j.png")
#定义一个5X5全是1的矩阵
kernel = np.ones((5,5),np.uint8)
#腐蚀图像
erosion = cv2.erode(img, kernel, iterations = 1)

cv2.imwrite("j2.png",erosion)
```
<img src="http://7xrooc.com1.z0.glb.clouddn.com/ercode.png">

### cv2.dilate
膨胀图像
```
img = cv2.imread("j.png")
#定义一个5X5全是1的矩阵
kernel = np.ones((5,5),np.uint8)
#膨胀图像
erosion = cv2.dilate(img, kernel, iterations = 1)
cv2.imwrite("j2.png",erosion)
```
### opening
开运算
开运算类似于腐蚀，应用于去除噪声
```
img = cv2.imread("j.png")
#椒盐
salt(img,200)
cv2.imwrite("j2.png",img)

#opening
kernel = np.ones((5,5),np.uint8)
opening = cv2.morphologyEx(img, cv2.MORPH_OPEN,kernel)

cv2.imwrite("j3.png",opening)
```
<img src="http://7xrooc.com1.z0.glb.clouddn.com/salt-open.png">

### closeing
闭运算
闭运算和开运算相反，连接被分开的块
```
opening = cv2.morphologyEx(img, cv2.MORPH_CLOSE,kernel)
```
### threshold
二值化
```
retval, result = cv2.threshold(src, thresh, maxval, type[, dst])
```
对应参数分别是：图片(灰度图)，阈值，最大值，类型
返回值分别为：阈值，结果图像
```
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('pic4.jpg',0)
ret,thresh1 = cv2.threshold(img,127,255,cv2.THRESH_BINARY)
ret,thresh2 = cv2.threshold(img,127,255,cv2.THRESH_BINARY_INV)
ret,thresh3 = cv2.threshold(img,127,255,cv2.THRESH_TRUNC)
ret,thresh4 = cv2.threshold(img,127,255,cv2.THRESH_TOZERO)
ret,thresh5 = cv2.threshold(img,127,255,cv2.THRESH_TOZERO_INV)

titles = ['Original Image','BINARY','BINARY_INV','TRUNC','TOZERO','TOZERO_INV']
images = [img, thresh1, thresh2, thresh3, thresh4, thresh5]

for i in xrange(6):
    plt.subplot(2,3,i+1),plt.imshow(images[i],'gray')
    plt.title(titles[i])
    plt.xticks([]),plt.yticks([])

plt.show()
```
自适应二值化，阈值会不断变化
cv2.ADAPTIVE_THRESH_MEAN_C的阈值是周围像素的灰度均值;
cv2.ADAPTIVE_THRESH_GAUSSIAN_C则是周围像素的高斯窗口加权和。
```
result = cv2.adaptiveThreshold(src, maxValue, adaptiveMethod, thresholdType, blockSize,C) 
```
对应参数分别为：图片(单通道)，最大值，自适应方式，二值化方式，窗口大小，常数C
先求窗口灰度均值，然后减去常数C作为阈值
返回结果为：目标图像
```
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('pic4.jpg',0)
img = cv2.medianBlur(img,5)

ret,th1 = cv2.threshold(img,127,255,cv2.THRESH_BINARY)
th2 = cv2.adaptiveThreshold(img,255,cv2.ADAPTIVE_THRESH_MEAN_C,\
            cv2.THRESH_BINARY,11,2)
th3 = cv2.adaptiveThreshold(img,255,cv2.ADAPTIVE_THRESH_GAUSSIAN_C,\
            cv2.THRESH_BINARY,11,2)

titles = ['Original Image', 'Global Thresholding (v = 127)',
            'Adaptive Mean Thresholding', 'Adaptive Gaussian Thresholding']
images = [img, th1, th2, th3]

for i in xrange(4):
    plt.subplot(2,2,i+1),plt.imshow(images[i],'gray')
    plt.title(titles[i])
    plt.xticks([]),plt.yticks([])
plt.show()
```
### bitwise_not
像素取反，两次取反回归原图
```
img = cv2.imread("pic.jpg")
cv2.namedWindow("Image")
result = cv2.bitwise_not(img)
cv2.imshow("Image", result)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
**blur**
低通滤波
```
res = cv2.blur(img,(5,5))
```
图片变得平滑（模糊）
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"显示一个图形"

import cv2
import numpy as np

img = cv2.imread("pic4.jpg")
res = cv2.blur(img,(5,5))

cv2.imshow("img",img)
cv2.imshow("res",res)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
左是处理后的图像，右是原图像
<img src="http://7xrooc.com1.z0.glb.clouddn.com/blur.png">
**boxfilter**
同样也是滤波，效果相同
第二个参数-1指的是图像深度由输入图像决定
```
res = cv2.boxFilter(img, -1, (5,5))
```
### 高斯模糊
```
res = cv2.GaussianBlur(img,  (5,5), 1.5)
```
### 中值滤波
```
res = cv2.medianBlur(img, 5)
```

# VideoCapture
## 摄像头
```
import numpy as np
import cv2

#创建一个VideoCapture对象
cap = cv2.VideoCapture(0)

while(True):
    # 一帧一帧读取
    ret, frame = cap.read()

    # Our operations on the frame come here
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # Display the resulting frame
    cv2.imshow('frame',gray)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# When everything done, release the capture
cap.release()
cv2.destroyAllWindows()
```
首先创建一个VideoCapture对象，参数0用于区别多个，通常只有一个摄像头连接，常用0或-1
```
ret, fram = cap.read()
```
ret是一个bool值，如果成功读取图像则返回True，可以通过检测ret的值判断是否读取成功
有时候，cap可能初始化失败，可以通过**cap.isOpened()**函数判断cap有没有初始化成功
**cap.get(propld)**函数，可以获得摄像头画面的一些参数
propld是0到18的数字，每个数字代表摄像头画面的一个性能参数(如果摄像头支持)
有些参数可以通过**cap.set(propld,value)**修改，value是你想自定义的新值

比如使用cap.get(3)和cap.get(4)，获得宽和高
```
cap = cv2.VideoCapture(0)
print cap.get(3),cap.get(4)
```
获得的是640 X 480，可以改成320 X 240
```
ret = cap.set(3,320)
ret = cap.set(4,240)
```
## 播放视频文件
和获取摄像头画面相似，只要把参数改为文件目录
当显示视频时，要注意**cv2.waitKey()**的时间参数(单位ms)，如果太短或者太长，视频将会变快或者变慢，正常设置为25ms
如果不能播放，确认已经安装了ffmpeg或者gstreamer
```
import numpy as np
import cv2

cap = cv2.VideoCapture("/home/herui/mv/badapple.mp4")

while(True):
    # Capture frame-by-frame
    ret, frame = cap.read()

    # Display the resulting frame
    cv2.imshow('frame',frame)
    if cv2.waitKey(25) & 0xFF == ord('q'):
        break

# When everything done, release the capture
cap.release()
cv2.destroyAllWindows()
```
## 保存视频文件
保存视频文件必须要指定文件名，FourCC，每秒的帧数(fps)以及每帧大小，还有isColor标识
FourCC是一个4-byte编码，用来指定视频编码器
FourCC使用cv2.VideoWriter_fourcc('M','J','P','G')或者cv2.VideoWriter_fourcc(*'MJPG')指定MJPG
```
import numpy as np
import cv2

cap = cv2.VideoCapture(0)

print cap.get(3),'*',cap.get(4)

#定义编码器，创建视频写对象
fourcc = cv2.VideoWriter_fourcc(*'XVID')
out = cv2.VideoWriter("output.avi", fourcc,25.0,(640,480))

while(cap.isOpened()):
    ret, frame = cap.read()
    if ret == True:
        #写帧
        out.write(frame)

        cv2.imshow('frame',frame)
        if cv2.waitKey(50) & 0xFF == ord('q'):
            break
    else:
        break

#release everything
cap.release()
out.release()
cv2.destroyAllWindows()
```
# matplotlib
## 简介
matplotlib是python的一个绘图库，具有和matlab相似的API
# 图像转换
## cv2 image to PIL image
```
img = cv2.imread("pic.jpg")
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
pil_img = Image.fromarray(img)
pil_img.show()
```
## PIL image to cv2 image
```
out = Image.open(src).convert("RGB")
cv_img = np.array(out)
cv_img = cv2.cvtColor(cv_img, cv2.COLOR_BGR2RGB)
```
# PIL filter
```
from PIL import Image,ImageFilter
 
#滤镜名称   含义
#ImageFilter.BLUR   模糊滤镜
#ImageFilter.CONTOUR    轮廓
#ImageFilter.EDGE_ENHANCE   边界加强
#ImageFilter.EDGE_ENHANCE_MORE  边界加强(阀值更大)
#ImageFilter.EMBOSS     浮雕滤镜
#ImageFilter.FIND_EDGES     边界滤镜
#ImageFilter.SMOOTH     平滑滤镜
#ImageFilter.SMOOTH_MORE    平滑滤镜(阀值更大)
#ImageFilter.SHARPEN    锐化滤镜

def filterDemo():  
    img = Image.open("pic4.jpg")  
    #img = inHalf(img)  
    imgfilted = img.filter(ImageFilter.SHARPEN)  
    #imgfilted.show()  
    imgfilted.save("pic4_1.jpg")  
  
if __name__ == "__main__":  
    filterDemo()  
```
# 参考
<http://blog.csdn.net/column/details/opencv-tutorial.html>
<http://blog.csdn.net/sunny2038/article/details/9057415>
<http://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_gui/py_video_display/py_video_display.html>
