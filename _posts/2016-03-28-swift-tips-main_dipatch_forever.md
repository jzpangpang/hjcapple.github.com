---
title: Swift tip：反复执行某段代码
layout: post
published: true
---

我用 dispatch 编写了一个小函数，可以方便地反复执行某段代码。

	func dipatch_forever(duration: Double, _ queue : dispatch_queue_t, _ block : dispatch_block_t)
	{
	    let time = dispatch_time(DISPATCH_TIME_NOW, Int64(duration * Double(NSEC_PER_SEC)))
	    dispatch_after(time, queue) {
	        block()
	        dipatch_forever(duration, queue, block)
	    }
	}
   
	func main_dipatch_forever(duration: Double, _ block : dispatch_block_t)
	{
	    dipatch_forever(duration, dispatch_get_main_queue(), block)
	}

每当我怀疑某界面或函数有问题。就随手写个回调，不断地 push 界面，pop 界面。或者不断地用随机数调用某些函数。或者不断创建删除对象。

	main_dipatch_forever(0.0) {
	    // Do something
	}

这样界面或函数中稍微有内存泄露，或者程序不够稳定。就会很容易看到内存飞快地涨或程序直接崩掉。经得起这种压力测试的，那段程序基本上都会很稳定（正确性需要额外测）。

另外当我想优化某段代码，也会将这段代码放入到 `main_dipatch_forever` 中反复调用，方便使用 Timer Profiler 测时间。 

这里不能使用死循环来测试，在 iOS 开发中，会产生一些对象加入到 Autorelease Pool 中。在一个事件循环周期内，这些对象是不会被释放掉的。这样测试就不准确了。
