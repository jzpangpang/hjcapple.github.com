---
title: iOS 开发中，搭建界面的一些争论
layout: post
published: true
---

开发 iOS 程序，搭建界面是避免不了的。到底应该怎么搭建界面呢？到现在还是一直争论不休。主要有两个争论：

* 使用 xib/storyboard 这样的图形化方式搭建界面，还是使用代码。
* 界面布局，是使用传统 frame 布局，还是使用 AutoLayout 设置约束。

上面两个争论是相互独立的。使用代码可以选择 frame 布局或 AutoLayout。使用 xib/storyboard，同样也可以选择 frame 布局或 AutoLayout。

但我发现，似乎很多人都搞不清楚这个相互独立的关系。他们会认为 AutoLayout 只能在 xib/storyboard 中使用，使用代码就只能用最原始的 frame 布局。从而有类似的问题，iPhone 有各种尺寸，是不是 iOS 已经不应该使用代码去手写界面了？

## frame 布局还是 AutoLayout

这个争论到今天算是慢慢消停了。iPhone 的屏幕已经有多种尺寸，应该优先采用 AutoLayout。只是 AutoLayout 也有缺陷，如：

* 没有占位符，导致一些界面需要使用看不见的 view 作为占位。
* 某些很常见的界面布局，比如等间距，用 AutoLayout 难以表述。
* 需要配置很多约束，而约束设置比较繁琐。
* 需要根据约束去求解真正的位置，某些场合下会有性能问题。比如稍微复杂一点的 TableViewCell。

分析某样东西的时候，应该同时分析优缺点，只要优点大于缺点，这样东西就是有价值的。AutoLayout 有缺陷，在某些场合下需要注意，但整体上看 AutoLayout 是很有价值了。

今天的 iOS 开发，界面布局应该是以 AutoLayout 为主，以传统的 frame 布局为辅助。但就算是使用传统的 frame 布局，也是有很多辅助库，并不用自己一个个去手算 frame。比如我自己就写过一个 [LayoutKit](https://github.com/hjcapple/LayoutKit)。

## xib 的历史

上面提到 xib/storyboard 等图形化方式，将 xib 和 storyboard 写在一起了。其实支持图形化方式的人，自身内部也是有个争论的：

* 使用多个独立的 xib，还是使用 storyboard。

在这里，我想先扯开一下，提一提 xib 的历史。

我们用图形化、鼠标拖拉来创建界面的这个工具，原本叫 Interface Builder。在 Xcode 4.0 之前，Interface Builder 是一个独立的软件。而 Xcode 4.0 是个大版本，界面被重新设计，Interface Builder 被直接集成到 Xcode。

Xcode 3.0 之前 Interface Builder 创建的文件是二进制格式 nib，nib 代表 NeXT Interface Builder。乔布斯从苹果公司出走之后，创建了下一个公司 NeXT, 之后 NeXT 又被苹果公司收购。现在 iOS 或 OS X 开发的整套系统、工具、类库最开始都源自 NeXT。二进制格式不好管理，也不方便版本控制，Xcode 3.0 之后，Interface Builder 使用了一种新的文件格式 xib。xib 的意思可能是 XML Interface Builder，也有可能是 OS X Interface Builder。xib 使用了 XML，在工程编译的时候再转换成 nib。

平时常说 xib 拼接界面。这里的 xib 根据上下文，通常并非指 xib 这种文件格式。而是使用 Interface Builder 这个工具。其实 xib 可以配置任何 NSObject 对象，但我们绝大多数情况下只用它来创建界面。

Interface Builder 的这段历史可以参考 [Mac OS X 背后的故事（五）Jean-Marie Hullot的Interface Builder神话](http://news.cnblogs.com/n/124292/)。

## storyboard

至于 storyboard，可以看成是一堆 xib 的集合，每个 xib 对于一个场景(scene)，并可设置场景之间的关系。场景的关系，在界面表示成一些连线，这样可以比较直观地看出它们的跳转，嵌套等关系。

因为 storyboard 可以看成一堆的 xib 集合，因此 xib 的所有缺陷，storyboard 也同样存在。另外 storyboard 的想法是好的，但相比较 xib 有额外的限制。

* storyboard 将多个场景同时显示出来，打开时候会比较慢。
* 电脑的屏幕较小时（比如我在用的的是 13 寸的 MacBook Pro)，编辑起来会很拥挤。
* storyboard 在多人编辑时，进行合并经常会冲突。

现实中使用 storyboard 的项目，经常会将 storyboard 分拆成多个。项目组的成员每人负责一个，尽量避免同时编辑 storyboard。

storyboard 中可以设置场景之间的关系，但是设置比较弱的。实际工程中的界面切换很多时候并非是线性的，而是根据条件来跳转。某条件下跳转到这个界面，另一条件下跳转到另一个界面，这样 storyboard 还是需要代码来配合。

另外做 iPhone 和 iPad 的 Universal 版本。会出现不同的设备下，界面布局有所不同。比如设置界面，iPad 屏幕较大，可以设计成分屏，ViewController1、ViewController2 左右并排。iPhone 屏幕较小，就设计成先显示 ViewController1，点击之后再显示 ViewController2。这种情况下，嵌套关系完全不同，用 Storyboard 不是很适当了。这样可以将 ViewController1，ViewController2 拆分分离的 xib，在用根据情况用代码将界面拼接处理。

storyboard 将多个场景同时显示出来，这既是它的优点，也恰好是它的缺点。

## xib/storyboard 等图形化方式，还是使用代码

我试过使用多个 xib，试过使用 storyboard，也试过使用代码。最后我采用的方案是：以代码为主，使用 xib 作为辅助，但不使用 storyboard。很多人可能不认同，先不要着急反驳，让我先说说我的理由。我也没有打算说服其它人，或者以后我的观点又会改变。

对于编程（包括界面编程），我有 3 个基本观念。

* 编程，应该是批量解决问题。
* 编程，应该追求复用。
* 编程，应该追求容易修改。

追求批量解决问题、并可以复用，这样你的工作就可以越做越快，可能最开始会慢一点，但会越来越快。

而一个项目随着时间推移，需求是肯定会变的。有些人追求预测准确，要一开始就去估计未来的变化。我相信有些人可以预测到将来的需求，只是这些人都是天才，我这种凡人根本就学不来。因此我追求的并非预测准确，而是当变化来临的时候，可以更容易修改。这种思路的不同，就会导致行为的不同。比如我不会花太多时间去定义需求，而只要有一个大方向就可以动手。这样我可以一开始写的很简单，之后再根据需求不断重构调整。

而使用 xib/storyboard 的图形化方式，经常会跟这三个基本观点相违背的。

xib/storyboard 不可以批量解决问题。比如我有 N 个界面，需要换字体颜色，这样就需要改动 N 次。

xib/storyboard 搭建的界面也不可以复用。粒度越大的东西越难复用，粒度越少的东西越容易复用。某个按钮，某个 view，某种基本布局，是很容易复用的，而xib/storyboard 拼接的大界面就难复用，并且 storyboard 比 xib 更难复用。

编程时候，一些有经验的人往往会将代码分拆比较细碎，再组合起来。就如同先生成一些小积木块，再拼接成一些大的积木块，一层层拼接上去。而在一些新手看来，这些代码会难以理解。新手倾向于一上来就做一个不可分解的一大块，觉得这样才是好的。新手往往会将好代码改坏了，而自己完全不知道。

至于容易修改，这个跟批量解决问题其实是有点重复的。假如修改时需要改动 N 次，本身就不容易修改。而就算是同一个界面，使用 xib/storyboard 配置约束之后，那些约束也是很难改动的。比如有 3 个按钮，配置好约束，但需要在中间插入一个按钮，改成 4 个按钮，往往再要折腾一阵子。另外接手他人的 xib/storyboard 文件进行修改，很难弄清楚它的约束关系，往往接手的时候往往会将约束全部删掉，再自己动手重新配置一遍。

xib/storyboard 用来验证想法做一些原型，或者做一些基本不会改动 Demo 时。但对于一个需要一直维护，经常改动的项目，会越来越慢。

因此我不太喜欢 xib/storyboard 等图形化方式来拼接界面，而使用代码。有时也用 xib 来辅佐。但我只用 xib 来设置控件的样式了，比如字体、颜色、文字之类，之后再用代码来设置 AutoLayout 的约束。

## 图形化

有些人总推荐图形化，而排斥写代码，他们觉得这样简单。但到底什么是简单呢？不少人认为简单就是直观，不用思考，一看就会了。而我自己认为，简单是指行为一致，绝少或者没有例外情况。对于我来说，简单的东西可以是不直观的，可能需要学习，但一旦学会了，就可以无差别地处理一大堆问题。

因此图形化的东西并非一定是简单，代码也并非一定是复杂的。

我并非反对图形化，只是不喜欢用鼠标在 xib 界面上拖拉约束。这样做很傻很慢，并很不灵活。我甚至觉得 xib 在 AutoLayout 时代，用鼠标拖拉约束，已经走入了歧途。

很多人想象中的图形化工具，就是不用写代码，这些工具生成大量的配置文件，就为了避免人们写代码。他们似乎觉得用写代码就是落后和原始的。但实际上在 iOS 界面开发中，写代码根本不是一个问题。最主要问题是，写代码到真正可以预览的时间太长了。更应该关注的是如何缩小代码到预览的反馈过程，而不是排斥代码。

对于程序员来说，代码反而是最直观，最灵活，最好控制的。

我更希望的方式，还是用代码来写界面，但可以实时看到最终效果。就好像做网页和写 Markdown 一样。我很希望 Swift 语言 playground 可以应用到实际工程当中，比如某个文件可以在编译和解释之间自由切换。平时开发的时候，是解释执行的，修改代码之后可以实时看到效果。最后发布的时候，是编译过的，这样可以加快运行速度。

图形化工具，目的并非是不用写代码，真正的目是缩短修改到预览的反馈过程。让我们眼睛可以实时看到变化，而不仅仅通过想象。

## AutoLayout 布局库

用代码手写约束是很麻烦的，所以就出现很多 AutoLayout 布局库。

假如 Objective-C 语言，推荐使用 [Masonry](https://github.com/SnapKit/Masonry)。

假如 Swift 语言，可以使用 [Cartography](https://github.com/robb/Cartography) 或者 [SnapKit](https://www.zhihu.com/question/42251996/answer/94998652)。

这几个库以前我都使用过，都十分出色。只是这些库用起来还是不够方便，它们的 API 每次调用只配置一两个 view 之间的约束，从代码本身很难从整体看到各个 view 之间的关系。

现在我使用的布局库，是用 Swift 语言自己编写的。有兴趣的可以看看这里 [AutoLayoutKit](https://github.com/hjcapple/AutoLayoutKit)。

## Size Class

现在 iOS 开发，提到界面适配，总会说 AutoLayout 的 Size Class，两个东西总被拿过来一起说。其实 AutoLayout 跟 Size Class 是两个完全不同的东西。

Size Class 其实只是按照 view 的尺寸大小，分成几个类型。之后就在 xib/storyboard 根据类型不同，分开设置两套不同的约束。比如同时适配 iPhone 和 iPad，就为 iPhone 这种 Size Class 类型设置一次约束，再为 iPad 设置一套约束。但 Size Class 的分类其实很不完整的，比如 iPhone 6 Plus 这种介乎于手机和平板之间的尺寸，就容易出问题。

而 Size Class 对于我这种用代码布局的人来说，根本就没有多大作用。我都已经用代码了，布局的时候往往也可以拿到 View 的精确大小，这样就没有什么必要再对 View 的尺寸大小进行分类。

有些人提到 xib/storyboard 的好处，也往往是说，用 AutoLayout 和 Size Class，可以一行代码都不写，就适配 iPhone 和 iPad。上文我就提到过，不写代码并非是个好处。界面逻辑不写在代码里面，就是在配置里面，肯定还是有个地方需要写的。只是写在配置里面，你看不到而已，复杂性还是一直在的。

