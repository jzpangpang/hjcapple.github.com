---
title: 开发的工具和库列表
layout: post
published: true

---

记录下一些开源软件，代码的使用。方便自己以后查找。

CocoaPods
--------
CocoaPods是一个负责管理iOS项目中第三方开源代码的工具。

参考 [使用CocoaPods来做iOS程序的包依赖管理](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/)

git地址 [https://github.com/CocoaPods/CocoaPods](https://github.com/CocoaPods/CocoaPods)


Tweaks
---------
微调iOS动画参数

参考 [Facebook发布Tweaks：让微调iOS应用变得更简单](http://techcrunch.cn/2014/03/26/facebooks-new-tweaks-library-lets-developers-fine-tune-their-ios-apps-on-the-fly/)

git地址 [https://github.com/facebook/Tweaks](https://github.com/facebook/Tweaks)


BlocksKit
---------
让一些iOS基础类，更好地支持Block语法。

git地址 [https://github.com/zwaldowski/BlocksKit](https://github.com/zwaldowski/BlocksKit)


Masonry
---------
定义了自己的DSL。大大简化iOS/Mac中，AutoLayout的写法。比如

	[view1 mas_makeConstraints:^(MASConstraintMaker *make) {
	    make.edges.equalTo(superview).with.insets(padding);
	}];
	
git地址 [https://github.com/cloudkite/Masonry](https://github.com/cloudkite/Masonry)


Classy
--------
使用类似css的方式，去配置UIView界面。

网址: [http://classy.as/](http://classy.as/)

git地址 [https://github.com/cloudkite/Classy](https://github.com/cloudkite/Classy)


Reveal
-----------
查看iOS应用的UI层次结构。

网址: [http://revealapp.com/](http://revealapp.com/)


MAObjCRuntime
--------
使用object-c语言封装 /usr/include/objc 头文件的函数。

git地址 [https://github.com/mikeash/MAObjCRuntime](https://github.com/mikeash/MAObjCRunt]ime)



