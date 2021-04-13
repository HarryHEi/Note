---
title: 限流算法
date: 2021-4-13 17:00:00
tags: [basic]
---

限流的目的是在访问超载时保证系统不被拖垮，超过负载的请求会被直接拒绝。

### 漏桶算法

顾名思义，漏洞算法就是新的请求会被放到一个漏桶中，之后匀速的处理漏桶中的请求。

参考: [`go.uber.org/ratelimit`](https://pkg.go.dev/go.uber.org/ratelimit)

#### 特点

请求处理的速度比较均匀，QPS很稳定，对于突发请求，很可能会直接拒绝。

#### 实现

在QPS固定的情况下，匀速的处理请求，本质上是给频率高的请求增加一个延时。

对于间隔比较近的多次请求，会强制性的要求请求等待一段时间处理，因此实现上需要有一个变量存储请求间的最小间隔时间，以及一个变量存储上次请求处理的时间。

同时，为了并发安全性，需要有个互斥锁。

因此，可以定义以下结构体

```
type LeakyRateLimitBucket struct {
	last       time.Time
	perRequest time.Duration

	mu sync.Mutex
}
```

+ `last`保存上次处理请求的时间

+ `preRequest`保存请求间的最小间隔时间，可以通过`1秒/QPS`获得
+ `mu`是互斥锁

定义一个初始化方法，计算最小间隔时间，返回限流器结构体

```
func NewLeakyRateLimitBucket(rate int64) *LeakyRateLimitBucket {
	return &LeakyRateLimitBucket{
		perRequest: time.Second / time.Duration(rate),
	}
}
```

下面定义`Take`方法用于执行权限获取

```
func (b *LeakyRateLimitBucket) Take() {
	b.mu.Lock()
	defer b.mu.Unlock()
	
	//...
}
```

对于第一次出现的请求，直接可以通过，同时记录本次时间

```
now := time.Now()

if b.last.IsZero() {
	b.last = now
	return
}
```

对于非第一次的请求，计算与上次请求处理时间的时间差，如果不满足最小间隔，则延时一段时间

```
sleepFor := b.perRequest - now.Sub(b.last)
if sleepFor > 0 {
	time.Sleep(sleepFor)
	// 这里如果用b.last = time.now()会有误差
	b.last = now.Add(sleepFor)
} else {
	b.last = now
}
```

使用测试

```
func testCustomLeakyBucketRateLimit() {
	bucket := NewLeakyRateLimitBucket(500)

  // 请求计数
	count := 0
	
	// 这里计算QPS
	ticker := time.NewTicker(time.Second)
	go func() {
		last := 0
		for range ticker.C {
			fmt.Println(count-last, " QPS")
			last = count
		}
	}()

  // 开启多个协成模拟请求
	for i := 0; i < 10; i++ {
		go func() {
			for {
				bucket.Take()
				count += 1
			}
		}()
	}
	
	select {}
}
```

输出

```
# go run main.go
500  QPS
499  QPS
500  QPS
499  QPS
500  QPS
```

### 令牌桶算法

想象有一个桶，匀速的往桶里面放token，每处理一个请求都需要从桶里面取一个token，如果token用完则拒绝请求。

参考: [`github.com/juju/ratelimit`](github.com/juju/ratelimit)

#### 特点

相比较漏桶算法，令牌桶算法可以处理突发请求。

#### 实现

和漏桶类似，也需要通过QPS计算间隔时间，不过这个间隔时间指的是往桶里面加token的时间。

其次，也必须有互斥锁。

此外，还需要有变量保存token总容量和剩余量。

结构体如下

```
type RateLimitBucket struct {
	fillInterval    time.Duration
	capacity        int64
	availableTokens int64
	last            time.Time

	mu sync.Mutex
}
```

+ `fillInterval`表示token填充的间隔时间，这里为了简化，每次填充一个token，因此可以通过`1秒/QPS`计算
+ `capacity`表示容量，容量越高，能够接受的突发请求数越多
+ `avaliableTokens`表示剩余可用的token数
+ `last`记录上次处理请求的时间，通过该时间在下次处理请求时判断填充的token数

定义一个初始化方法，计算填充间隔时间，返回限流器结构体

```
func NewRateLimitBucket(rate int64, capacity int64) *RateLimitBucket {
	return &RateLimitBucket{
		fillInterval:    time.Second / time.Duration(rate),
		capacity:        capacity,
		availableTokens: capacity,
	}
}
```

通过上次填充的时间和填充间隔可以计算需要填充的token周期数，因为每个周期填充1，因此同时也是填充的token数

```
func (b *RateLimitBucket) currentTick() int64 {
	return int64(time.Now().Sub(b.last) / b.fillInterval)
}
```

如果token还有剩余，则不进行填充，否则填充相应数量的token，每次请求都需要进行token容量调整

```
func (b *RateLimitBucket) adjustAvailableTokens() {
	// 剩余token足够时不做操作
	if b.availableTokens >= b.capacity {
		return
	}

	tick := b.currentTick()
	// 剩余token减少后根据经过的周期数增加token，这里倍率是1，每个周期加1 token
	b.availableTokens += tick
	if b.availableTokens > b.capacity {
		b.availableTokens = b.capacity
	}
	return
}
```

定义一个token消费的方法，如果token数量充足，则正常消费，返回true，否则返回false，在这里会调用调整容量的方法

```
func (b *RateLimitBucket) IsAvailable() bool {
	b.mu.Lock()
	defer b.mu.Unlock()
	// 第一次请求直接通过
	if b.last.IsZero() {
		b.last = time.Now()
		return true
	}

	b.adjustAvailableTokens()
	avail := b.availableTokens - 1
	if avail > 0 {
		b.availableTokens = avail
		b.last = time.Now()
		return true
	}
	return false
}
```

使用测试

```
func testCustomTokenBucketRateLimit() {
	bucket := NewRateLimitBucket(500, 500)

  // 请求计数
	count := 0

  // QPS计算
	ticker := time.NewTicker(time.Second)
	go func() {
		last := 0
		for range ticker.C {
			fmt.Println(count-last, " QPS")
			last = count
		}
	}()

  // 开启多个协程模拟请求
	for i := 0; i < 10; i++ {
		go func() {
			for {
				take := bucket.IsAvailable()
				if take {
					count += 1
				}
			}
		}()
	}

	select {}
}
```

输出

```
# go run main.go
998  QPS
500  QPS
500  QPS
500  QPS
```

可以看到第一次统计QPS是大于设置的值的，因为token桶容量是满的，可以看出只要容量足够，令牌桶算法可以处理突发的流量。

