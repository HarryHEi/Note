---
title: Unity Transform
date: 2018-2-9 21:54:37
tags: [unity]
---


对象的位置、旋转和缩放。

场景中的每个对象都有一个Transform，存储对象的位置、缩放和旋转。

# public method
## transform.Rotate

public void Rotate(Vector3 eulerAngles, Space relativeTo=Space.Self);

eulerAngles 旋转角度
relativeTo 相对对象旋转还是全局旋转
