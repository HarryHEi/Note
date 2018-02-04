---
title: timer
date: 2017-08-09 15:00:55
tags: [c_plus]
---

# 简介

C++中实现延时以及计时功能

# 标准库

在标准库中，头文件`"time.h"`或者用C++风格`"ctime"`头文件下有一个`clock()`函数。

函数会返回一个从CRT初始化开始经过的时间，除以`CLOCKS_PER_SEC`得到单位为秒的值。


可以以此实现一个延时函数。
```
void do_sleep(clock_t wait)
{
	clock_t goal;
	goal = wait + clock();
	while (goal > clock());
}

int main(int argc, char *argv[])
{
	do_sleep((clock_t)3 * CLOCKS_PER_SEC);

	return 0;
}
```

或者计算一段程序运行的时间

```
int main(int argc, char *argv[])
{
	clock_t begin_t, end_t;

	begin_t = clock();
	do_sleep((clock_t)3 * CLOCKS_PER_SEC);
	end_t = clock();

	std::cout << "elapsed time: " 
		<< (double)(end_t - begin_t) / CLOCKS_PER_SEC 
		<< std::endl;

	return 0;
}
```

# boost
## timer

在"boost/timer.hpp"中利用clock()实现了一个timer类，用来对程序进行计时。时间跨度为几百小时，不适合较大时间跨度的情况。

```
int main(int argc, char *argv[])
{
	boost::timer t;

	do_sleep((clock_t)3 * CLOCKS_PER_SEC);

	std::cout << "elapsed time: " << t.elapsed() << std::endl;

	return 0;
}
```

## progress_timer

头文件"boost/progress.hpp"，继承自timer，析构时会自动调用`elapsed()`打印输出。
初始化时可以指定输出流，默认为std::cout。

```
void func()
{
	boost::progress_timer t;
	do_sleep((clock_t)3 * CLOCKS_PER_SEC);
}

int main(int argc, char *argv[])
{
	func();

	return 0;
}
```

## progress_display

同样在头文件"boost/progress.hpp"中，是一个独立的类，用于显示百分比进度。

初始化的时候传入一个基数，然后调用"+="方法增加基数。

在遍历一个数组的同时，显示进度的代码入下：
```
int main(int argc, char *argv[])
{
	std::vector<int> vi{ 1,2,3,4,5,6,7,8,9,10};
	int sum = 0;
	boost::progress_display pd(vi.size());

	for (auto v: vi)
	{
		do_sleep((clock_t)v + CLOCKS_PER_SEC);
		sum += v;
		++pd;
	}

	return 0;
}
```

输出结构像这样：
```
0%   10   20   30   40   50   60   70   80   90   100%
|----|----|----|----|----|----|----|----|----|----|
***************
```

注意的是，如果程序也有输出的话，格式会变得混乱，除非调用restart()重新显示刻度。
