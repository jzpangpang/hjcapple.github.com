---
title: Swift tip: 保证 Int 或者 string 的唯一
layout: post
published: true
---

编程中时候常常需要定义一些 Int 常量或者 String 常量。数字常数可以作为 Index 或 tag。比如：

	private let kVersionCellIndex = 0
	private let kSettingsCellIndex = 1
	......
	private let kCloseButtonTag = 1000
	private let kOpenButtonTag  = 2000
	
字符串常量可以作为字典的 key。比如：

	private let kDefaultEmailKey = "defaultEmail"
	private let kFontSizeKey = "fontSize"
	
这种情况下，需要保证常量的值唯一。

最近遇到一个小 Bug, 我定义了一些字符串常量，作为 NSUserDefaults 的 key。之后需要新增一项，很自然地复制粘贴了一下，修改成：

	private let kDefaultEmailKey = "defaultEmail"
	private let kFontSizeKey = "fontSize"
	xxxxxx
	private let kDefaultThemeKey = "defaultEmail"
	
kDefaultThemeKey 就是新增的，但是我忘记修改后面的字符串值了。这样写 kDefaultThemeKey 时候就修改了 kDefaultEmailKey 对应的内容。

为防止以后发生类似的事情，我想到一个小技巧，将字符串常量定义成 enum：

	private enum Key : String
	{
	    case defaultEmail         = "defaultEmail"
	    case defaultTheme         = "defaultTheme"
	    case fontSize             = "fontSize"
	    xxxxx
	}
	
这样假如忘记修改值了，编译时候就会告诉我错了，就可以保证常量的唯一，Int 也是一样。再定义一个小函数:

	setValue(value: AnyObject forKey key: Key)
	{
		let strKey = key.rawValue
		xxxxx
	}

这样就不用总是写成 `Key.defaultEmail.rawValue`、`Key.fontSize.rawValue`。

