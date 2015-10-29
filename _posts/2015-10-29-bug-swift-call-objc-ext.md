---
title: Bug 备忘：Swift 调用 Objective-C 的静态库中的扩展函数
layout: post
published: true
---

其实这个应该不算 Bug, 是我配置有错误。但也作个备忘。

我将原来工程的 Objective-C 代码，做成一个静态库，让 Swift 的工程调用。发现一但调用了扩展函数，就会运行错误。

比如 Objective-C 的 NSString 有个扩展函数

	@interface NSString (ToolkitAddtions)
	
	// 生成全局唯一字符串
	+ (NSString*)UUIDString;
	
	@end
	
Swift 中调用

	let UUID = NSString.UUIDString()
	
运行时就出错，出错信息为

	+[NSString UUIDString]: unrecognized selector sent to class 0x10d124b20
	
后来查了查资料，是我 Swift 的工程配置有误。需要在`Other Linker Flags` 加上 `-Objc`。
