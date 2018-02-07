---
title: Unity Input
date: 2018-2-7 23:52:37
tags: [unity]
---

Input类用于管理输入，包括控制器、手柄、键盘、鼠标的传统输入，还有移动设备的触摸屏和移动感应功能，还支持VR和AR系统的输入，此外还有麦克风和设想有的音频和视频数据。

如果使用输入进行移动控制，使用`Input.GetAxis`，操作平滑，可以映射到键盘、鼠标或者控制器。`Input.GetButton`用于执行类似事件的操作，不要用于控制移动，`Input.GetAxis`更小更简单。

例如获取轴信息，可以是手柄的轴或者键盘的方向键或w、a、s、d:
```
float moveHorizontal = Input.GetAxis("Horizontal");
float moveVertical = Input.GetAxis("Vertical");
```
