---
title: 开发的工具和库列表
layout: post
published: true

---

记录下一些开源软件，代码的使用。方便自己以后查找。

[CocoaPods](https://github.com/CocoaPods/CocoaPods)
--------
CocoaPods是一个负责管理iOS项目中第三方开源代码的工具。

参考 [使用CocoaPods来做iOS程序的包依赖管理](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)

<br>


[Tweaks](https://github.com/facebook/Tweaks)
---------
微调iOS动画参数

参考 [Facebook发布Tweaks：让微调iOS应用变得更简单](http://techcrunch.cn/2014/03/26/facebooks-new-tweaks-library-lets-developers-fine-tune-their-ios-apps-on-the-fly/)


[BlocksKit](https://github.com/zwaldowski/BlocksKit)
---------
让一些iOS基础类，更好地支持Block语法。


[Masonry](https://github.com/cloudkite/Masonry)
---------
定义了自己的DSL。大大简化iOS/Mac中，AutoLayout的写法。比如

	[view1 mas_makeConstraints:^(MASConstraintMaker *make) {
	    make.edges.equalTo(superview).with.insets(padding);
	}];
	

[Classy](https://github.com/cloudkite/Classy)
--------
使用类似css的方式，去配置UIView界面。


[Reveal](http://revealapp.com/)
-----------
查看iOS应用的UI层次结构。


[MAObjCRuntime](https://github.com/mikeash/MAObjCRunt]ime)
--------
使用object-c语言封装 /usr/include/objc 头文件的函数。


[Crashlytics](http://try.crashlytics.com/)
--------
收集程序的崩溃信息。


[CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)
----------
一个 Mac 和 iOS 可用的日志库


[LXReorderableCollectionViewFlowLayout](https://github.com/lxcid/LXReorderableCollectionViewFlowLayout)
------
扩展 UICollectionViewFlowLayout，支持类似 iBooks, Flipboard 那样的书架拖动效果。


[facebook/Shimmer](https://github.com/facebook/Shimmer)
-----
添加类似Facebook-Paper中，那种波光闪动的效果。


[facebook/chisel](https://github.com/facebook/chisel)
-------
LLDB 的增强工具。用于辅助调试 iOS app。



[facebook/KVOController](https://github.com/facebook/KVOController)
--------
用于 key-value 编程



