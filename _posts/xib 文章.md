---
title: iOS 界面开发
layout: post
published: false
---

开发 iOS 程序，搭建界面是避免不了的。Xcode 包含了一个工具 Interface Builder，可以用图形化的方式，使用鼠标拖拉来创建界面。

Xcode 3.0 之前 Interface Builder 创建的文件是二进制格式 nib，nib 代表 NeXT Interface Builder。乔布斯从苹果公司出走之后，创建了下一个公司 NeXT, 之后 NeXT 又被苹果公司收购。现在 iOS 或 OS X 开发的整套系统、工具、类库最开始都源自 NeXT。二进制格式不好管理，也不方便版本控制，Xcode 3.0 之后，Interface Builder 使用了一种新的文件格式 xib。xib 的意思可能是 XML Interface Builder，也有可能是 OS X Interface Builder。xib 使用了 XML，在工程编译的时候再转换成 nib。

在 Xcode 4.0 之前，Interface Builder 是一个独立的软件。而 Xcode 4.0 是个大版本，界面被重新设计，Interface Builder 被直接集成到 Xcode 当中，也将默认编译器从 gcc 切换到 LLVM。

其实 Interface Builder 可以配置创建任何 NSObject 对象，但我们绝大多数情况下只用它来创建界面。

开发 iOS，到底怎么搭建界面，一直到现在也争论不休，主要是两个争论。

* 使用图形化拖拉 vs 使用代码。
* Frame 布局 vs AutoLayout。

而图形化方式，又有一个争论。

* 使用多个独立的 xib vs 使用 storyboard。

上面的关系是相互独立的。比如使用代码，也可以选择 frame 布局或 AutoLayout。而图形化方式，也不一定使用 AutoLayout。有些人搞不清楚这个关系，总会觉得使用代码不能用 AutoLayout，只能用原始的 frame 布局。

另外上面的关系，并非是完全对立的，完全可以配合起来。同一个工程中，一部分界面可以使用 xib, 一部分界面使用代码。或一部分界面使用 frame 布局，一部分界面使用 AutoLayout。

至于 Size Class，其实也是一个独立的部分，只是很多文章都会将 Size Class 和 AutoLayout 放到一起说。

## Storyboard

Storyboard 可以看成是一堆 xib 的集合，每个 xib 对于一个场景(scene)，并可设置场景之间的关系。场景的关系，在界面表示成一些连线，这样可以比较直观地看出它们的跳转，嵌套等关系。

Storyboard 的想法是好的，只是 Storyboard 有自己的限制。

* Storyboard 将多个场景同时显示出来，打开时候会比较慢。
* 电脑的屏幕较小（比如我在用的的是 13 寸的 MacBook Pro)，编辑起来会很拥挤。
* Storyboard 多人编辑后，进行合并会经常冲突。

通常的解决方法是拆分 Storyboard，将其拆分成多个，项目组的成员每人负责一个，尽量避免同时编辑 Storyboard。

Storyboard 中可以设置场景之间的关系，但是设置是很弱的。实际工程中的界面切换很多时候并非是线性的，而是根据条件来跳转。某条件下跳转到这个界面，另一条件下跳转到另一个界面，这样就需要代码来配合。

另外做 iPhone 和 iPad 的 Universal 版本。会出现不同的设备下，界面布局有所不同。比如设置界面，iPad 屏幕较大，可以设计成分屏，ViewController1、ViewController2 左右并排。iPhone 屏幕较小，就设计成先显示 ViewController1，点击之后再显示 ViewController2。这种情况下，嵌套关系完全不同，用 Storyboard 不是很适当了。这样可以将 ViewController1，ViewController2 拆分分离的 xib，在用根据情况用代码将界面拼接处理。

Storyboard 将多个场景同时显示出来，这既是它的优点，也恰好是它的缺点。

## Xib


/////////

在一个实际工程中，需求是会不断修改。对于与代码界面


我个人习惯是用代码写UI的。代码写界面有些好处：
代码容易复用。
容易批量调整，比如修改颜色的配置，全部界面颜色都变了。

也有人喜欢storyboard，好处也很明显：
直观，方便跟设计师一起调整界面。
可以很快地拼接出一个App原型。（但假如界面变动，就很惨了，几乎要重新拼过）。

我的看法是：
短期快速出一个Deom来验证想法，xib或storyboard会很快。但长期来看，用代码写UI未必会慢。
假如你屏幕越大，xib或storyboard就越有优势；屏幕越小，代码就写UI就越有优势。

我以前用iMac时，也倾向于用xib或storyboard。后来用了MacBook Pro，就改成写代码了。

到了AutoLayout时代，我个人觉得用鼠标在storyboard中拖那些约束实在太麻烦了，就转成使用Cartography库来手写布局。而有些几乎不变的布局，我就关掉xib的AutoLayout，直接拖拉，比如一些弹出的消息框。

用代码或者storyboard做UI哪个好，一直在争论的。不同团队选择会有不同，多些选择也挺好啊，看你个人喜欢啦。假如加入一个团队，对这方面有规定，就以团队的标准来做，反正转过来也很容易。

storyboard的问题是，将很多界面都放在同一个文件中。这样多人协同开发时候，就会修改同一个文件。再用版本控制工具合并时，容易做成冲突。
最后一点，我有所误导。根据评论的说法，可以切分storyboard。




