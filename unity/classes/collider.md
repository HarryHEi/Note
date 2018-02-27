---
title: Unity Collider
date: 2018-2-9 22:27:37
tags: [unity]
---

Collider(碰撞机)

当一个game object具有某个碰撞机，比如一个正方体碰撞机，那么它与其他对象接触时会表现的像个正方体。

# Messages

void OnTriggerEnter(Collider other);

当有其他物体进入附加了刚体的对象，将调用这个函数。
