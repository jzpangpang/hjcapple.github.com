---
title: C++ 编程规范（6）-- 函数
layout: post
published: true
---

<a name="short_function"></a>
## 6.1 编写短小的函数
函数尽可能的短小，凝聚，功能单一。

只要某段代码，可以用某句话来描述，尽可能将这代码抽取出来，作为独立的函数，就算那代码只有一行。最典型的就是C++中的max, 实现只有一句话。

	template <typename T>
	inline T max(T a, T b)
	{
		return a > b ? a : b;
	}

* 将一段代码抽取出来，作为一个整体，一个抽象，就不用纠结在细节之中。
* 将一个长函数，切割成多个短小的函数。每个函数中使用的局部变量，作用域也会变小。
* 短小的函数，更容易复用，从一个文件搬到另一个文件也会更容易。
* 短小的函数，因为内存局部性，运行起来通常会更快。
* 短小的函数，也容易阅读，调试。

<a name="less_than_five"></a>
## 6.2 函数的参数可能少，原则上不超过5个
人脑短时记忆的数字是很有限的，大约可以记忆7个数字。有些人多些，有些人少些。我们这里取最少值，就是5个参数。

参数的个数，太多，就很容易混乱，记不住参数的意义。

同时参数的个数太多，很可能是因为这个函数做的事情有点多了。

可以通过很多手段来减少参数的个数。比如将函数分解，分解成多个短小的函数。或者将几个经常一起的参数，封装成一个类或者结构。比如，设计一个绘画贝塞尔曲线的接口

	void drawQuadBeizer(float startX,   float startY,
						float controlX, float controlY,
						float endX,     float endY);

这样的接口，就不够

	void drawQuadBeizer(const Point& start,
	                    const Point& control,
	                    const Point& end);

简洁易用。

当然，每个规则都会有例外。比如设置一个矩阵的数值，二维矩阵本来就需要6个数字来表示，设置接口自然需要6个参数。

<a name="order"></a>
## 6.3 函数参数顺序
参数顺序，按照传入参数，传出参数，的顺序排列。不要使用可传入可传出的参数。

	bool loadFile(const std::string& filePath, ErrorCode* code);  // 对
	bool loadFile(ErrorCode* code, const std::string& filePath);  // 错

保持统一的顺序，使得他人容易记忆。


<a name="use_pointer"></a>
## 6.4 函数的传出参数，使用指针，而不要使用引用
比如

	bool loadFile(const std::string& filePath, ErrorCode* code);  // 对
	bool loadfile(const std::string& filePath, ErrorCode& code);  // 错

因为当使用引用的时候，使用函数的时候会变成

	ErrorCode code;
	if (loadFile(filePath, code))
	{
		...
	}

而使用指针，调用的时候，会是

	ErrorCode code;
	if (loadFile(filePath, &code))
	{
		...
	}

这样从，&code的方式可以很明显的区分，传入，传出参数。试比较

	doFun(arg0, arg1, arg2);    // 错
	doFun(arg0, &arg1, &arg2);  // 对

<a name="default_args"></a>
## 6.5 不建议使用函数的缺省参数
我们经常会通过查看现有的代码来了解如何使用函数的接口。缺省参数使得某些参数难以从调用方就完全清楚，需要去查看函数的接口，也就是完全了解某个接口，需要查看两个地方。

另外，缺省参数那个数值，其实是实现的一部分，写在头文件是不适当的。

缺省参数，其实可以通过将一个函数拆分成两个函数。实现放到.cpp中。

