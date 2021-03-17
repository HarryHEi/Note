---
title: sync.map
date: 2020-10-21 9:23:00
tags: [golang]
---

golang 标准库中的map不是原子操作，如果多个协程同时读写会抛出异常。`sync.Map`通过`CAS`(CompareAndSwap)和锁实现协程安全。

### 源码

`sync.Map`结构如下

```
type Map struct {
	mu sync.Mutex

	read atomic.Value
	dirty map[interface{}]*entry
	misses int
}

type readOnly struct {
	m       map[interface{}]*entry
	amended bool 
}
```

`read`：指向一个只读结构体（包含一个`map`和一个`amended`标识），保存一部分数据，用来处理只读操作，因此不需要加锁。

`dirty`：一个`map`，保存另一部分数据，用于加锁的读写操作。

因此`sync.Map`内部含有两个`map`，一个只用于读操作，一个支持写操作，这样设计是为了减少加锁的次数。

### 读

首先先检查key是否在`read`中，如果有的话直接返回，这个过程不会加锁。

如果`read`中没有找到，则加锁，然后从`dirty`中查询，同时增加`misses`计数，如果数值大于`dirty`中元素的总数，`dirty`中的元素会被拷贝到`read`，`dirty`清空，并且重置计数，最后返回值。为了避免过多的访问`derty`，`read`中会保存一个额外字段`amended`来表示`dirty`中是否有值，只有在该字段为`True`时才会查询`dirty`。

因此，大部分情况下，读操作非常高效，几乎和普通的`map`相同，针对从一个大`map`中获取数据的情况，`sync.Map`很有效。

### 更新

首先会检查`read`，如果找到值得话会通过`atomic.CompareAndSwap`更新值。

如果`read`中没有找到，会对`dirty`进行同样的动作，这里会检查`dirty`有没有初始化，如果没有则初始化。

所以，更新值是比较简单的，但是新增数据比较复杂。

### 迭代

在迭代之前，如果`dirty`中有值，会加锁并把`dirty` 中的数据拷贝到`read`，锁释放之后使用迭代普通的`map`的方式迭代。

### 总结

`sync.Map`是包含两个`map` 的复杂结构，不仅仅是`map`加锁，针对查询做了优化。在同时有读写操作时，`sync.Map`的`read`和`dirty`中都会有数据，这种情况下，`sync.Map`的表现比`map`加锁的方式要差（不仅要检查两个`map`，而且要更新计数器）。

