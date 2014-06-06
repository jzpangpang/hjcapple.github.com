---
title: 开发的工具和库列表
layout: post
published: true

---

记录下一些开源软件，代码的使用。方便自己以后查找。

## 工具

| 名称            	| 说明
|:--------------		|--------
| [CocoaPods] 		| 管理iOS项目中第三方开源代码。
| [facebook/Tweaks] | 微调iOS动画参数 
| [Reveal] 			| 查看iOS应用的UI层次结构。
| [Crashlytics] 		| 收集程序的崩溃信息。
| [facebook/chisel] | LLDB 的增强工具。用于辅助调试 iOS app。

[CocoaPods]: 			https://github.com/CocoaPods/CocoaPods
[facebook/Tweaks]: 	https://github.com/facebook/Tweaks
[Reveal]: 			http://revealapp.com/
[Crashlytics]: 		http://try.crashlytics.com
[facebook/chisel]: 	https://github.com/facebook/chisel


## 源码

| 名称            | 说明
|:--------------		|--------
| [ReactiveCocoa]	| 函数式，响应式编程框架 
| [BlocksKit] 		| 让一些iOS基础类，更好地支持Block语法。
| [Masonry] 			| 定义了自己的DSL。大大简化iOS/Mac中，AutoLayout的写法。
| [Classy] 			| 使用类似css的方式，去配置UIView界面。
| [MAObjCRuntime] 	| 使用object-c语言封装 /usr/include/objc 头文件的函数。
| [CocoaLumberjack] | 一个 Mac 和 iOS 可用的日志库
| [LXReorderableCollection<br>ViewFlowLayout] | 扩展 UICollectionViewFlowLayout，<br>支持类似 iBooks, Flipboard 那样的书架拖动效果。
| [facebook/Shimmer] 		| 添加类似Facebook-Paper中，那种波光闪动的效果。
| [facebook/KVOController] | 用于 key-value 编程

[ReactiveCocoa]: https://github.com/ReactiveCocoa/ReactiveCocoa
[BlocksKit]: 		https://github.com/zwaldowski/BlocksKit
[Masonry]: 		https://github.com/cloudkite/Masonry
[Classy]: 		https://github.com/cloudkite/Classy
[MAObjCRuntime]: https://github.com/mikeash/MAObjCRuntime
[CocoaLumberjack]: https://github.com/CocoaLumberjack/CocoaLumberjack
[LXReorderableCollection<br>ViewFlowLayout]: https://github.com/lxcid/LXReorderableCollectionViewFlowLayout
[facebook/Shimmer]: https://github.com/facebook/Shimmer
[facebook/KVOController]: https://github.com/facebook/KVOController


## 常用命令

| 命令            | 说明
|:--------------		|--------
| qmake -spec macx-xcode	| 从 qt 配置文件，生成 xcode工程



