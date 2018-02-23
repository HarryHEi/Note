---
title: MonoBehaviour
date: 2018-2-8 23:45:37
tags: [unity]
---


MonoBehaviour是每个脚本派生类的基类，当使用`C#`时必须显式继承MonoBehaviour。

# MonoBehaviour.Start()

在第一次调用Update之前调用。

和启动函数类似，Start()只会被调用一次，不同的是，如果脚本未始能，将不会调用Start()。

启动函数在所有场景对象的Start()调用之前调用。

# MonoBehaviour.Update()

每一帧被调用，间隔时间不固定，注意如果用于物体的移动或者摄像机跟随，将不会平滑移动。

使用`Time.deltaTime`可以获取直到上次调用经过的时间。

这个函数只有在Behaviour是enable状态才会调用(MonoBehaviour或者Component的enable属性)。

# MonoBehaviour.FixedUpdate()

每一帧被调用，调用时间间隔固定。当处理`Rigidbody`时应该使用FixedUpdate而不是Update。

# MonoBehaviour.LateUpdate()

在所有Update调用结束后再调用，用于调整脚本的执行顺序，适合用于相机跟随。

# Messages
## Awake

当脚本实例加载后调用。

用于在游戏开始前初始化变量和游戏状态。只会调用一次。在所有对象初始化后调用。
