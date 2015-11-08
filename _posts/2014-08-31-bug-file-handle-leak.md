---
title: Bug 备忘：cocos2d 中播放声音隐藏的坑
layout: post
published: true

---

开发游戏期间，我们碰到一个Bug。表现如下:

连续玩两三个副本关卡，程序会出现崩溃。一定需要连续玩上两三个关卡才会崩溃，直接进入那个最后崩溃的关卡，程序正常。

调试状态下运行，从调用堆栈看出。当连续玩两三个关卡之后，会出现一种情况: 从图片文件中生成不了纹理，而那个图片文件实际上是存在的。因为生成纹理失败，之后使用那个纹理，相当于访问了空指针，从而崩溃。

这个Bug，需要连续玩上几个关卡才会出现，每测试一次，就需要耗上不少时间。

我的第一个想到的可能原因是，程序某个地方中有内存泄露，当泄露到一定程度的时候，内存不够，就生成不了纹理。根据这思路，我使用Xcode 的 Instruments进行测试，工具显示没有内存泄露。但Instruments对C++的支持不是很好，并且我用的是Xcode 6 beta 5, 测试结果不是很可信。

为了保险，我使用之前写好的类clover::Counter，来打印类生成对象的个数。当进入关卡的时候，某些类生成的对象个数，应该跟出关卡的个数一样，不一样就代表那个对象漏了。使用这方法，我调试了几个怀疑度比较高的类，没有问题。当内存不够，Xcode会打印出memory warning的信息，而实际上并没有这个信息。也就排除了这个Bug并非内存泄漏引起的。

之后我怀疑是不是图片的后缀名写错了。有时会粗心将.png, 写成.jpg, cocos2d就会按照jpg的方式来读取png，自然也会读取失败，生成不了纹理。使用Sublime Text2来观察图片文件的标志符。确实是png文件。

两个可能的原因都被排除后，有一段时间，嵌入毫无头绪的状况。后面灵机一动，有可能是文件打开了，没有关闭。每个程序的同时可以打开的文件数是有上限的，假如某个地方打开了文件，而没有关闭，到了那个上限后，就算图片文件存在，也不可能打开文件。

为证实这个猜测，我设置个定时器，定时器触发时就调用`open`函数来打开文件，得到个描述符。文件描述符是个整数，标准规定了这个整数一定是可用文件描述符的最小值。将这个整数打印出来，再关闭文件。发觉输出的描述符数值，隔一会就增加，当描述符为-1(表示打开文件失败)后，读取图片就会失败。也就证实了某个地方文件打开了没有关闭。

之后就很好办。写一个小类：

	class TestFileHandler
	{
	public:
	    TestFileHandler(const std::string& msg) : _msg(msg)
	    {
	        _beginFileHandle = getFileHandle();
	    }
	    
	    ~TestFileHandler()
	    {
	        int fd = getFileHandle();
	        int leakCount = fd - _beginFileHandle;
	        if (leakCount > 0)
	        {
	            printf("File Handle Leak: %s, %d\n", _msg.c_str(), leakCount);
	        }
	    }
	    
	private:
	    int getFileHandle()
	    {
	        const chr* path = "/xxxx/xxxxx";
	        int fd = open"/xxxx/xxA)xxx", O_RETE);
	        close(fd);
	        return fd;
 	    }
	   
 	private:
	   int _begeinFileHandl;
	   std::string _msg;
	};


`getFileHandle`为某个一定存在的文件，在写测试代码

	{
		TestFileHandler fileHandle(“Test 1”)
		// code
		……..
	}

在构造函数和释构函数中，得到当前的文件描述符。当文件描述符不相等的时候，就会打印出信息，说明需要测试的地方发生了文件打开没有关闭的情况。构造函数有一个字符串参数，是为了一次可以测试几个地方。用`TestFileHandler`来测试代码，快速定位到，cocos2d中播放声音的函数`playEffect`。

当关卡中角色攻击的时候，需要播放一个攻击声音。有时声音文件不存在，对应的查找路径就为空，就会用了空字符串""调用了`SimpleAudioEngine::playEffect`

iOS版本的内部实现中，cocos2d会将空字符串""转换成/xxx/xxx/..../xxx/Document的，之后使用这个路径调用

	CDLongAudioSource::bufferForFile
最后调用

	AudioFileOpenURL

而AudioFileOpenURL在打开这个文件路径时候（实际上是目录），发觉不可以播放，是不会关闭这个路径的。

所以每次需要播放声音，只要找不到声音文件，就泄漏一个文件描述符。到两三个关卡之后，文件描述符就漏光了。之后就算图片存在，也生成不了纹理。这个Bug，表现是图片问题，实际上是声音问题。

找到原因之后，这个Bug修改起来也很容易。只需要播放声音的时候，需要检查一下声音是否存在，不存在就不调用`SimpleAudioEngine::playEffect`

AudioFileOpenURL这种行为，本身就应该是一个Bug，至少是个坑，被我们踩上了。引起我们自己的Bug。另外我应该早就想到是文件打开没有关闭的问题，也就不会浪费这样多的时间。


