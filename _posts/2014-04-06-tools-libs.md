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
| [JASidePanels] | 左右滑动，露出侧边栏
| [RectangleBinPack] | 二维装箱问题
| [MGSwipeTableCell] | TableViewCell左右滑动，露出下面的按钮
| [nanovg] | 轻量级矢量图绘画库

[ReactiveCocoa]: https://github.com/ReactiveCocoa/ReactiveCocoa
[BlocksKit]: 		https://github.com/zwaldowski/BlocksKit
[Masonry]: 		https://github.com/cloudkite/Masonry
[Classy]: 		https://github.com/cloudkite/Classy
[MAObjCRuntime]: https://github.com/mikeash/MAObjCRuntime
[CocoaLumberjack]: https://github.com/CocoaLumberjack/CocoaLumberjack
[LXReorderableCollection<br>ViewFlowLayout]: https://github.com/lxcid/LXReorderableCollectionViewFlowLayout
[facebook/Shimmer]: https://github.com/facebook/Shimmer
[facebook/KVOController]: https://github.com/facebook/KVOController
[JASidePanels]: https://github.com/gotosleep/JASidePanels
[RectangleBinPack]: https://github.com/juj/RectangleBinPack
[nanovg]: https://github.com/memononen/nanovg
[MGSwipeTableCell]: https://github.com/MortimerGoro/MGSwipeTableCell


## 常用命令

| 命令            | 说明
|:--------------		|--------
| qmake -spec macx-xcode	| 从 qt 配置文件，生成 xcode工程
| stars:>1	| 在 github 中 搜索有星星的项目，之后可以按照星星多少来排序
| svn propset svn:ignore "*.obj" . | 设置svn的忽略文件
| svn propedit svn:ignore . | 使用编辑器来修改svn的忽略文件

## Mac 系统常用快捷键
| 快捷键           | 说明
|:--------------		|--------
| Command + W | 关闭窗口
| Command + Q | 关闭程序
| Command + Space | 切换输入法
| Command + Control | Spotlight
| Command + Tab | 切换程序
| Command + S | 保存
| Command + F | 搜索
| Command + Option + F | 替换



