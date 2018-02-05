---
title: 颜色识别
date: 2016-04-20 20:28:34
tags: [python]
---

# 功能
识别红色和蓝色

# 用到的库

+ opencv
+ numpy

思路是找寻红色或者蓝色的边界，然后画上矩形。

# 代码如下
```
#!/usr/bin/env python
# -*-coding:utf-8 -*-

import numpy as np
import cv2

def find_red_and_blue(img):
    height, width, channel = img.shape
    rows= []
    cols = []
    #色调H∈ [0, 180）， 饱和度S ∈ [0, 255]， 亮度V ∈ [0, 255]
    #H:
    #   Orange  0-22
    #   Yellow 22- 38
    #   Green 38-75
    #   Blue 75-130
    #   Violet 130-160
    #   Red 160-179
    HSV_img = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

    flag = 0

    for row in range(height):
        for col in range(width):
            H = HSV_img[row][col][0]
            S = HSV_img[row][col][1]
            V = HSV_img[row][col][2]

            # blue \  red 根据实际情况做了微调
            if (H >= 160 and H <= 179 and  V >= 90 and V <= 255 and S >= 90 and S <= 255) or \
                (H >= 111and H <= 130 and  V >= 150 and V <= 255 and S >= 150 and S <= 255):
                rows.append(row)
                cols.append(col)
                flag = 1

    # 没有检测到红色或者蓝色
    if flag == 0:
        return 0,0,0,0
    row_max = max(rows)
    row_min = min(rows)
    col_max = max(cols)
    col_min = min(cols)

    return row_max,row_min,col_max,col_min

#打开摄像头
cap = cv2.VideoCapture(0)

#缩小图片
ret = cap.set(3,160)
ret = cap.set(4,120)

while True:

    ret, img = cap.read()

    height, width, channel = img.shape

    row_max,row_min,col_max,col_min = find_red_and_blue(img)

    # 画矩形
    try:
        if row_max != 0 and height/(row_max - row_min) <= 100 and width/(col_max - col_min) <= 100:
            pts = np.array([[col_min, row_min], [col_min, row_max], [col_max, row_max], [col_max, row_max]], np.int32)
            pts = np.array([[col_min,row_min],[col_max,row_min],[col_max,row_max],[col_min,row_max]], np.int32)
            pts = pts.reshape((-1,1,2))
            img = cv2.polylines(img,[pts],True,(0,255,255))

     #除零错误
    except ZeroDivisionError:
        pass

    #img = cv2.resize(img, (320,240), interpolation=cv2.INTER_CUBIC)
    # Display
    cv2.imshow('image',img)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

#release everything
cap.release()
cv2.destroyAllWindows()
```