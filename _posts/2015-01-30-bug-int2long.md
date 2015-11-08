---
title: Bug 备忘：64 位平台，int* 强转 long* 
layout: post
published: true
---

开发完成后，将 .ipa 包提交到苹果商店，审核通过。之后又发现一个 Bug。

在 64 位的 iOS 设备上面玩我们的游戏，使用内购来购买钻石，之后再做其它的操作，游戏会卡死。这个 Bug 只会出现在 64 位的设备上面，并且使用了内购之后才出现。32 位的机子没有问题。

经过一番调试，最终定位到问题所在。

	// 基础信息
	struct SimpleAny
	{
	    SimpleAny(int* aIntValue) : intValue((long*)aIntValue){};
	    SimpleAny(long* aIntValue) : intValue(aIntValue){};
	    SimpleAny(std::string* aStringValue) : stringValue(aStringValue){};
	    
	    long* intValue = nullptr;
	    std::string* stringValue = nullptr;
	};
	
	struct
	{
	    QueryUserData::DataType type;
	    SimpleAny field;
	    std::string notification;
	} updateInfos[] = {
	    { QueryUserData::DataType::UserID,   &_ID,     ""                              },
	    { QueryUserData::DataType::Nickname, &_name,   AppNotification::UpdateUserData },
	    { QueryUserData::DataType::Sign,     &_sign,   AppNotification::UpdateUserData },
	    { QueryUserData::DataType::Avatar,   &_avatar, AppNotification::UpdateUserData },
	    
	    // ...

见上面代码，其中 `SimpleAny` 的构造函数中，将 `int*` 转成 `long*`，之后将指针保存下来。其实将 `int*` 强转 `long*` 本来就有问题，我只负责调 Bug, 这样做的真实原因我也不太清楚。猜测可能是程序本来是针对 32 位编写的，程序当中顺便混用了 `int` 和 `long`。当最后需要改成 64 位时，为了可以快速编译通过，就不管三七二十一，直接加多个构造函数，将 `int` 强转成 `long`。

接下来的代码，就是根据 `QueryUserData::DataType` 的值来跟对应的指针赋值。

在 64 位平台上，int 是占 4 字节，long 占 8 字节，将 `int*` 被强转成 `long*`。这样再使用`long*`来写入数据，就有可能将原来数据后面的数据也覆盖掉。

![int2long](/media/images/bug-int2long.png)

假如我定义了

	int x; 
	int y;

上图是在小尾机子上的内存布局。本来 y 只占蓝色框的大小，但强转成 long* 之后，就被当成红色框的大小，这样当写入的时候，就冲掉了 x 的数据。

在 32 位的情况下，int 大小是跟 long 一样的，从而就没有问题。下面是个简化例子

	include <stdio.h>
	
	int x = 0; 
	int y = 1;
	
	static void setValue(long* a) 
	{
	   *a = 1;
	}
	
	int main(int argc, const char* argv[]) 
	{
	   for (x = 0; x < 10; x++)
	   {
	       setValue((long*)&y);
	       printf("%d\n", y);
	   }
	   printf("End\n");
	   return 0;
	}

这段代码，使用 32 位编译运行，正常退出。64 编译运行上，y 冲掉了 x 的数据，就形成死循环。

这样的 Bug 比较难以测试。有时就算冲掉了数据，假如数据没有被怎么用到，也看起来正常。而内购正好触发了之前埋下的隐患。另外有基本的内存概念，这样的 Bug 是一开始就可以避免的。
