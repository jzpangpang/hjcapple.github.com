---
title: 编写调试 QuickLook 插件
layout: post
published: true

---

开发者可以编写QuickLook插件，在OS X系统支持自定义的格式。

QuickLook插件可以放在三个目录下面。

* ~/Library/QuickLook/
* /Library/QuickLook/
* /System/Library/QuickLook/

系统会根据次序依次检测相应的支持插件。QuickLook的整个编写过程，苹果的官方文档也说得很清楚。不太清楚的是调试部分，毕竟文档有点旧了。

我自己摸索出的方式如下：

在Xcode工程里面配置`Per-configuration Build Products Path`配置，使编译出来的插件直接放在

	~/Library/QuickLook/
	
里面。再设置编译完成后的运行的脚本，每次编译完成执行：

	qlmanage -r
	
这命令使得插件生效，或者还需要加上命令

	qlmanage -r cache

我不知道怎么使用断点来调试插件，只是将调试信息直接打印到文件当中。这样也足够了。

最开始的时候，我不知道上面两条命令可以使插件生效。每次测试，都是先注销账号，再重新登录一次。这样就太费时了。


 

