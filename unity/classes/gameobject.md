---
title: Unity GameObject
date: 2018-2-9 22:38:31
tags: [unity]
---


场景中所有物体的基类

# properties
## GameObject.tag

public string tag;

游戏对象的标记，用于识别一个游戏对象。使用之前必须声明一个标签: Edit -> Project Setting -> Tags and Layers。

# Public Methods
## CompareTag

public bool CompareTag(string tag);

查看游戏对象是否有tag标记。

## SetActive

public void SetActive(bool value);

value: 是否激活

激活或者取消激活游戏对象。
