---
title: 使用 swift 编写的 view 布局库：LayoutKit
layout: post
published: true
---

这是我写的布局库 LayoutKit，README 文档的节选，工程已放到 [GitHub](https://github.com/hjcapple/LayoutKit) 上。

## AutoLayout 的问题
iOS/OS X 开发中为了适应多种尺寸，会使用 AutoLayout。但 AutoLayout 有些问题：

* 没有占位符。导致一些界面使用一些看不见的占位 view。
* 一些很常见的界面布局，比如等间距，用 AutoLayout 难以表述。
* 需要配置很多约束，就算使用 [Masonry](https://github.com/SnapKit/Masonry) 或 [Cartography](https://github.com/robb/Cartography) 等布局库，约束设置也比较繁琐。而在 xib 或者 storyboard 中拖拉约束，简直是反人类。（我承认这条原因比较主观。）

比如
### 界面1

<img alt="example0" src="/media/images/layoutkit/example0.png"/ width="320">

这种布局在 TableViewCell 中很常见，左边是一张图片，中间是字体不同的 label，几个 label 整体需要上下居中，最靠右是时间标签。

这种布局用 AutoLayout 表示，通常需要将几个 label 放到一个看不见的 view 上面。

### 界面2

<img alt="example1" src="/media/images/layoutkit/example1.png"/ width="320">

三个色块代表三个按钮，已经知道大小，需要左右排列。并且使得间距 1、2、3、4 相等。这种等间距布局也很常见，而用 AutoLayout 配置起来就十分麻烦。

### 另外
AutoLayout 需要动态求解约束，有小小的性能问题。在一些复杂的滚动列表界面，cell 中出现太多不必要的占位 view, 并且有太多约束，快速滚动时可能不够流畅。

为此基于传统的 frame 布局，编写了这个 LayoutKit。

我没有否认 AutoLayout 的作用。一些场合，比如多语言文字自动适配大小，基于各个子 view 的大小自动算出父 view 的大小。这种传统的 frame 布局是做不到的。

AutoLayout 和 frame 布局可以结合起来使用，各取优缺点。

## 使用例子
使用 LayoutKit 实现上述两个界面布局，使大家有个初步的认识。

### [界面1](#ui_1)

```Swift
self.tk_layoutSubviews { make in
    // 1
    let iconHeight = make.yFlexibleValue(10, make.flexible, 10)
    // 2
    make.size(iconView) == (iconHeight, iconHeight)
    // 3
    make.sizeToFit(titleLabel, detailLabel, longDetalLabel, timeLabel)
    // 4
    make.xPlace(10, iconView, 10, titleLabel, make.flexible, timeLabel, 10)
    // 5
    make.ref(titleLabel).xLeft(detailLabel, longDetalLabel)
    // 6
    make.yCenter(iconView, timeLabel)
    // 7
    make.yPlace(make.flexible, titleLabel, 6, detailLabel, longDetalLabel, make.flexible)
}
```
	
1. 算出 icon 的高度。
2. 设置 icon 的大小。
3. 设置各个 Label 的大小。
4. 水平排列 icon、titleLabel、timeLabel，并设置好各自的间距。
5. 根据 titleLabel 左对齐 detailLabel、longDetalLabel。
6. iconView、timeLabel 垂直居中。
7. 垂直排列 titleLabel、detailLabel、longDetalLabel，并居中。

### [界面 2](#ui_2)

```Swift
self.tk_layoutSubviews { make in
    // 1
    make.size(redView, blueView, greenView) == (80, 80)
    
    let F = make.flexible
    // 2
    make.xPlace(F, redView, F, blueView, F, greenView, F)
    // 3
    make.yCenter(redView, blueView, greenView)
}
```
    
1. 设置各个 view 的大小。
2. 水平排列 redView、blueView、greenView，并设置好各自的间距。
3. 各个 view 垂直居中。

## 概览

界面布局，大致分解成 3 步：

1. 设置大小。
2. 水平方向排列各个 views。
3. 垂直方向排列各个 views。

LayoutKit 提供 API 来设置大小、水平位置、垂直位置。水平垂直在英文中用 h 和 v 表示，但我实在是记不清楚。因此设定用数轴的概念：

* 水平方向，就是 x 方向。
* 垂直方向，就是 y 方向。

API 设计中

* xLeft、xRight、xCenter、yPlace 等等就是设置 x 方向。
* yTop、yBottom、yCenter、yPlace 等等就是设置 y 方向。

<a name="example0"></a>
很多界面布局库用起来繁琐，是它每次只操作一两个 view，但事实上我们更关心界面的整体布局。LayoutKit 将所有的 views 作为一个整体，一次排列多个 views，比如：

```Swift
make.xPlace(20, redView, make.flexible, blueView, 20)
```
	
这个调用，就出现下列布局:

<img alt="example2" src="/media/images/layoutkit/example2.png"/ width="320">

<a name="flexible"></a>
### flexible

LayoutKit 利用 flexible 占位符，来动态计算布局距离。根据总长度减去布局中出现的长度，剩余的长度被 flexible 占位符平分。

比如[上面例子](#example0)，假设总宽度为 375，redView 和 blueView 的宽度都为 100，这样 flexible 就计算得出：

```Swift
flexible = 375 - (20 + 100 + 100 + 20) = 135
```
	
也可以出现多个 flexible，比如

```Swift
make.xPlace(make.flexible, redView, blueView, make.flexible)
```
	
这代码就使得 redView 和 blueView 靠在一起，x 方向居中，而

```Swift
let F = make.flexible
make.xPlace(F, redView, F, blueView, F)
```
	
剩余空间被 3 个 flexible 均分，就产生均匀间距的布局。

另外 flexible 还可以乘以一个数字，比如 `make.flexible * 2`，就相当于 2 个 flexible。比如

```Swift
let F = make.flexible
make.xPlace(F, redView, F * 2, blueView, F)
```
	
也可以乘以一个小数，比如：
	
```Swift
let F = make.flexible
make.xPlace(F * 0.5, redView, F, blueView, F * 0.5)
```
	
flexible 可以用在计算高度和宽度，比如：
	
```Swift
make.height(redView, blueView, yellowView) == [44, make.flexible, 44]
make.xEqual(redView, blueView, yellowView)
make.yPlace(redView, blueView, yellowView)
```
    
将 blueView 的高度设置成 flexible，就产生下面界面：

<img alt="example3" src="/media/images/layoutkit/example3.png"/ width="320">

<a name="bounds"></a>
### 布局 bounds

布局 bounds 表示 view 需要放置在哪里，是一个用于参考的矩形。比如左对齐，就隐含了左边界在哪里。flexible 的值也是根据 bounds 的高度和宽度来计算的。

刚开始时，布局 bounds 等于父 view 的 bounds, 当也可以通过一些 API 去修改。比如：

```Swift
make.insetBounds(edge: 30)
make.equal(blueView)
    
make.insetBounds(edge: 30)
make.equal(redView)
```
    
bounds 先插入边距，放置 blueView, 再插入边距，再放置 redView, 就产生下面布局： 

<img alt="example4" src="/media/images/layoutkit/example4.png"/ width="320">

也可以分别设置上下左右的边距，之后通过 resetBounds 来还原 bounds，比如：

```Swift
do {
    let oldBounds = make.insetBounds(top: 10, left: 20, bottom: 40, right: 50)
    defer {
        make.resetBounds(oldBounds)
    }
    // do something
}
```

