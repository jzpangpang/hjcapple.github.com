---
title: Swift tip：在用 iOS 模拟器调试时，执行额外代码
layout: post
published: true
---

在开发 iOS App 过程中，经常需要在 iOS 模拟器调试时，执行一些额外代码。而在真机运行时，需要将这些代码去掉。

比如，

* 在模拟器启动时，打印出 Document 的目录。
* 访问网络时，打印出获取的数据，或者将网页写到 Mac 的某个文件，查看数据对不对。
* 在模拟器执行时，监听工程中某个文件内容的变化，当变化时直接刷新界面。

为达到上述目的，我们可以加入一些宏判断：
	
	#if (arch(x86_64) || arch(i386)) && os(iOS)
	    doSomething()
	#endif
	
iOS 真机的指令是 arm。因此当指令为 x86_64 或者是 i386, 并且 os 为 iOS，就表示在 iOS 模拟器中运行了。

上述宏判断是很难记住。编程过程中，假如需要反复纠结某段代码，就将其封装起来。我们编写一个简单的小函数：

	public func on_ios_simulator(@noescape fun: () -> Void)
	{
	    #if (arch(x86_64) || arch(i386)) && os(iOS)
	        fun()
	    #endif
	}

就可以这样使用：

    on_ios_simulator {
        let doc = DocumentPath()
        print(doc)
    }

当在真机运行时，on_ios_simulator 就是个空函数。编译器可以直接优化掉。

再举个例子：
	
	on_ios_simulator {
		let htmlPath = ProjectRes("version.html")
		watch_file(htmlPath) {
			loadHtmlPath(htmlPath)
		}
	}
	
上面这段代码表示当在 iOS 模拟器运行时，就会监听工程资源中 version.html 文件的变化。一但有变化就重新载入页面，这样就可以立即看到修改后的效果。



