---
title: Swift tip：在 Framework 中使用桥接头文件
layout: post
published: true
---

我现在编写 iOS 和 Mac 程序，绝大部分都采用 Swift 了，越来越少使用 Objective-C。但现存很多库是使用 Objective-C 编写的。这样不可避免遇到 Swift 和 Objective-C 的混编问题。

假如编译目标是 App，让 Swift 调用 Objective-C 会很简单。只需要在工程中设置一个桥接头文件，之后在这个头文件中包含 Objective-C 的头文件，就可以让 Swift 调用了。

但新建一个 framework，设置桥接头文件后。编译时会出错：

	using bridging headers with framework targets is unsupported
	
有个取巧方法可以解决这个问题。

当你创建了 framework，名字为 sharelib，在工程中会有个 sharelib.h 的文件，内容类似这样：

	#import <UIKit/UIKit.h>
	
	//! Project version number for sharelib.
	FOUNDATION_EXPORT double sharelibVersionNumber;
	
	//! Project version string for sharelib.
	FOUNDATION_EXPORT const unsigned char sharelibVersionString[];

将这个文件直接从工程中删掉（不是注释代码，而是直接删掉）。或者在 sharelib 的配置 Build Phases/Headers 中，将 sharelib.h 去掉。之后就一切跟在目标为 App 一样，配置好桥接头文件就行了。

这样做为什么可以呢，其实我也不清楚。我只是觉得既然在 App 可以使用桥接头文件，在 framework 中没有理由不可以，一定是某个配置有所不同。之后我就建立目标分别为 App、静态库、framework 的工程，依次对比它们的配置选项，再尝试修改。一个小时左右，发现了这个似乎很白痴的取巧方法。
	
更多的混编问题，可以参考这篇文章 [如何在IOS的Framework中使用Swift和Object-C进行混编](http://bravedefault.com/test/)。