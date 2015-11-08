---
title: Bug 备忘：Swift 中两个 UInt64 数据相减引起崩溃
layout: post
published: true
---

App访问服务器拉取消息，每个消息都有个属性为createTime，类型为Unix Timestamp，表示消息的产生时间。在App中展示消息界面时，需要显示消息的产生时间。比如显示：

* 1天前
* 10小时前
* 10分钟前
* 5秒前

产生时间距离现在的时间超过1天，就用天为单位；不超过1天，但超过1小时就用小时为单位；不超过1小时，但超过1分钟就用分钟为单位。以此类推。

很自然地，需要将手机端当前时间跟消息产生时间相减。

	func setupWithMessage(message: api.Message)
	{
	    let currentTimestamp = Timestamp(NSDate().timeIntervalSince1970) // (1)
	    let seconds = currentTimestamp - message.createTime              // (2)
	    
	    xxxxxx 根据时间差显示 xxxxx
	}
	
Timestamp定义为：

	typealias Timestamp = UInt64    // unix时间戳
	
上面程序看起来似乎没有什么问题，但当App获取新消后，在Timestamp相减那行，也就是第（2）行App会崩掉。崩溃信息为：

	Thread 1: EXC_BAD_INSTRUCTION (code=EXC_I386_INVOP,subcode=0x0) 
	
但崩溃之后，重新启动App再测试，就又正常了。每次想复现bug，都需要手动在服务器后台产生一条新的消息。
	
通常这种跟时间有关，一时一个样的Bug，会跟线程有关系。我以为message在别的线程或其它地方不小心被释放了，再次访问message的数据自然就崩掉了。但仔细检查调试了很久，也没有发现问题。最后可以完全复现问题：

	let seconds1 = message.createTime - Timestamp(0) // (1)
	let seconds2 = Timestamp(0) - message.createTime // (2)
	
(1)不会崩溃，(2)肯定会崩溃。我觉得这样会崩溃很神奇，而同事看了看，说这里可能整数溢出了。但按照以前的经验，就算溢出也不应该崩溃。我用下面代码试了试，真的会崩掉。

	let a: UInt64 = 1234
	var b = UInt64(1) - a
	
另一个同事发来网页，[Swift中文教程（二）基本运算符](http://letsswift.com/2014/06/basic-operators/)有这段话：

> 不同于C和Objective-C，默认情况下Swift的算术运算符不允许值溢出。你可以通过Swift的溢出运算符来选择值的溢出情况(例如 a & + b)。详见 [Overflow Operators](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AdvancedOperators.html#//apple_ref/doc/uid/TP40014097-CH27-XID_37)

这小段话，就算我之前看过，不遇到实际问题也不会注意到。

之后很容易想明白了bug出现的原因：

Timestamp定义成了Uint64。当手机端当前Timestamp比消息产生的Timestamp要大，两个Uint64相减，就没有问题。但当手机端当前Timestamp要小，就产生溢出，程序就崩溃了。

消息先产生，再被手机端获得，为什么手机端的Timestamp反而比消息Timestamp要小呢？这是因为服务器的时钟比手机端时钟快了一点点。假如消息发生时，服务器时间是13:40:50秒，而手机端时间是13:40:45秒。当通过网络取得消息时，手机端时间是13:40:46秒，这样手机端Timestamp自然比消息Timestamp要小了。当App崩溃之后重新启动，手机端时间变为13:41:30秒，之前会引起崩溃的消息重启之后不会再引起崩溃。

知道原因之后，修改起来也很容易，需要检查是否会溢出。

	func setupWithMessage(message: api.Message)
	{
		let currentTimestamp = Timestamp(NSDate().timeIntervalSince1970)
		var seconds = 0
		if currentTimestamp > message.createTime
		{
		    seconds = currentTimestamp - message.createTime
		}
		    
		xxxxxx 根据时间差显示 xxxxx
	}
	
另外将Timestamp重新定义为

	typealias Timestamp = Int64    // unix时间戳

可能会产生运算的地方，最好还是用有符号数字。其实数据出现溢出本身也是个错误，可以在调试时将Timestamp改成UInt64，而在最终发布的时候为求保险将Timestamp修改成Int64。这样就算不小心再次溢出，最多只是信息显示错误。


	

	
	
	

	


	