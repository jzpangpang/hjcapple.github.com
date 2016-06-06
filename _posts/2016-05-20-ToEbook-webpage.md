---
title: ToEbook 的宣传页
layout: post
published: true
---

做了 [ToEbook](http://dumbduck.github.io/ToEbook/) 的宣传页。就这一个简单的静态页面，前后花了三天时间。最开始我不熟悉做网页的流程、工具、库。下次做类似的东西会快些吧。

编辑器用 Sublime Text 3，装了个格式化 html/css 的插件。

做网页最大好处是编辑完一刷新就可以看到效果。最开始是手动刷新，后来发觉刷新太频繁了，就所搜索自动刷新工具。先试了 LiveReload，可能设置有问题，用不了。后来安装了 Firefox 的一个插件 AutoReload。

现在我的机器上有 Safari、Chrome、Firefox、Opera 四个浏览器。用 Firefox 来自动刷新查看效果，Chrome 用来调试并查看其它网页的结构。Chrome 本来是很好的，只是在中国 Google 的服务都被墙了，想安装 Chrome 插件也难。现在默认浏览器是 Firefox。

最开始很难理解 css 的布局方式，边查资料边试。

最后要做一个翻页效果。最开始查看 Apple 产品翻页效果源码，太多代码了看不懂。就上 GitHub 查找 JavaScript 的滚动库，先试了 [onepage-scroll](https://github.com/peachananr/onepage-scroll)，不是很适合。最后用了 [iscroll](https://github.com/cubiq/iscroll)。

我现在还没有买服务器，这个简单的静态页面就放到 GitHub Pages 上了。

还有些问题，需要调整，以后再弄了。

* 在 iPhone 上查看页面的最终效果，不是很好，拉得太长了。在电脑上面看，效果要好些。
* 这个宣传页有 5 张图，载入时候有点慢，需要进行优化，比如延迟加载。（已完成）
* 拖拉浏览器窗口大小时，宣传图会晃动。（已完成）

