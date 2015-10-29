---
title: Bug 备忘：静态缓冲区引起游戏闪退
layout: post
published: true

---

将《风暴传说》移植到 Android 的过程中，出现了个 Bug。游戏在 Android 机上玩两三个小时会偶然闪退。

进行调试，发现表面原因。

FlashFire 动态将 bin 文件的矢量数据合并成一张大图，生成纹理缓存，在某文件夹中写入 plist 文件和 png 图片。在通过某个 plist 文件读取纹理的过程中发生闪退。再检查那个 plist 文件，发现其中含有一个非法字符。plist 文件是 xml格式，在 Android 平台上，含有非法字符就导致 xml 格式分析错误，cocos2dx 本身并没有对读取失败的情况进行容错，从而闪退。

而在 iOS 上使用系统自带的函数来解析 plist，可能系统自身对一两个非法字符的情况进行了容错处理，使得本来不正确的 plist 文件也读取成功。因此 iOS 上没有出现闪退的bug。

根本问题是，为什么生成的 plist 文件偶然会出现一两个非法字符？

最后定位到错误的代码。

	static std::string intToString(int value)
	{
	    static char buf[32];
	    sprintf(buf, "%d", value);
	    return buf;
	}

这段代码，使用了静态缓冲区。使用静态缓存区的具体原因忘记了，有可能是为了加快速度，也有可能是手误。

最开始FlashFire是不支持多线程的，上面代码没有问题。后来 FlashFire 支持预加载，在额外的线程中预先生成纹理缓存。这样就会偶然两个或多个线程同时调用 `intToString`，导致字符缓存区混乱，从而生成一两个非法字符。这个 Bug 存在了很久，但最近移植到 Android 的过程中，才暴露出来。

修改的过程也很简单，缓存区去掉 static 修饰就可以了。

	static std::string intToString(int value)
	{
	    char buf[32];
	    sprintf(buf, "%d", value);
	    return buf;
	}
	
我犯了一个很低级的错误。

为了更保险，将根据 plist 文件载入纹理的过程重新编写了一次，加入了容错处理。使得纹理缓存就算有错误，也仅仅重新生成一次，不会闪退。


