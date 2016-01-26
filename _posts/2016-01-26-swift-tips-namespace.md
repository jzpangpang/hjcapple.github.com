---
title: Swift tip：使用 struct 来模拟命名空间
layout: post
published: true
---

Swift 没有类似 C++ 的命名空间。它分 module，一个 module 一个命名空间。实际工程中，每个 module 对应一个 framework target。而 target 太多，工程会难以管理。

因此我很希望 Swift 可以方便地在同一个 module 中划分命名空间，最好在语言上添加上 namespace 关键字。这个需求暂时可以使用 struct 去模拟。

比如工程中的网络部分，我想放到 network 命名空间中。可以定义

	struct network
	{
		class Connect
		{
			xxxxx
		}
	}
	
其它的文件，就使用 extension 语法

	extension network
	{
		class HttpConnect
		{
			xxxxx
		}
	}
	
这样就可以用名字 network.HttpConnect 来访问代码。假如嫌前缀太长，使用时候在 swift 文件中定义

	private typealias HttpConnect = network.HttpConnect
	
这样基本可以达到命名空间的作用。但我还是希望这种机制集成到语言中。

使用 struct 来模拟命名空间，而不是用 class。是因为在 Swift 中，struct 比 class 更加轻量。

