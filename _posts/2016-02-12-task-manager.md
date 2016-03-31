---
title: 我使用的任务管理方法
layout: post
published: true
---

我利用 TodoList 和番茄时间进行任务管理。

之前我试过很多 TodoList 软件。后来还是觉得 OS X 和 iOS 自带的提醒事项最简单最方便。

当需要做一个比较大的事情时，比如需要写一个工程，或者读一本书，就建立一个列表。比如做工程 Project-1，就建立一个 Project-1 的列表。先写下最开始想到的事项，比如最开始列表有：

* 建立 Xcode 工程。
* 建立 Tabbar 界面：主页、设置。
* 图标使用 svg。

有一个 Doing 的列表。Doing 表示正在做的事项。可以同时进行有多个项目，但 Doing 永远只有一个大项。当做这个大项时候，再将大项拆分成小项。比如：

* 图标使用 svg
* +++++++++++++++++++
* 调查有什么 svg 的库。
* 工程集成 svg 库。
* 找到一张测试用的 svg 图片。
* 将 svg 图片转成 UIImage。
* 建立资源目录。

有了初步列表项之后，就可以开始计时。用番茄时间，每工作 25 分钟，休息 5 分钟。再工作 25 分钟，再休息 5 分钟。

事项做完就勾掉。做这些事项的时候，会有出现新的事项，就在列表后面添加。

* 建立资源目录。
* +++++++++++++++++++
* 重构 xxx 代码。
* 需要发送一封邮件。

在 25 分钟之内，列表进行的顺序尽量不要改变。25 分钟之后，有 5 分钟休息时间。这 5 分钟内可以调整列表顺序，或者将新增的列表项重新归类。

这种简单方式可以使注意力集中到一件事情上。

## 时间追踪

我写了一个很简陋 iPhone 小软件给自己用来计时。

比如去吃饭，看书，或者看一个电影。我就开启计时器。我可以同时开启多个计时器。比如吃饭和看电影同时计时，就表示我一边吃饭一边看电影。或者一边坐车一边听播客。

计时停止之后，数据同步 OS X 和 iOS 自带的日历。我建立了很多小日历项。比如

* 工作
* 学习
* 运动
* 家务
* 休息
* 交通
* 个人
* 交流
* 娱乐

其中工作、学习、运动、家务这几个日历项是绿色的。 休息、交通、个人是灰色的。娱乐是红色的。

比如读书的分类为学习，看电影分类为娱乐，吃饭、洗澡等算个人。当我要做一件事情，比如上班（算交通）、看书、写程序等就开始计时。

计时同步到日历之后，一周后查看，我就知道今周具体做了什么事情。我根本不需要很精确的时间统计，只需要看看日历的颜色。假如这周的绿色比较多，我这周就比较勤快。假如红色比较多，我这周就是在偷懒了。

我还在慢慢开始养成这种计时习惯。不过很多时候也忘记计时。忘记计时了，日历就是空白的。