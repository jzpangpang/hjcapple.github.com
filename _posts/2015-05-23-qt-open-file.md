---
title: Bug 备忘：OS X 系统上，Qt 程序双击图标打开文件会崩溃
layout: post
published: true

---

OS X 系统上，在编写一个应用程序，想增加功能，双击文件的图标就可以直接打开应用程序。但是一直启动不了，一但双击文件图标，程序就会崩溃。而只是单纯的打开程序，不会崩溃。

双击打开，崩溃时候，出现的信息栈如下

	Thread 0 Crashed:: Dispatch queue: com.apple.main-thread
	0   libsystem_c.dylib             	0x00007fff97381152 strlen + 18
	1   org.qt-project.QtCore         	0x0000000105e60107 QCoreApplication::arguments() + 199
	2   libqcocoa.dylib               	0x0000000107c83eec 0x107c61000 + 143084
	3   com.apple.AppKit              	0x00007fff8ae71878 -[NSApplication(NSAppleEventHandling) _openDocumentURLs:withCompletionHandler:] + 875
	4   com.apple.AppKit              	0x00007fff8ae70729 __69-[NSApplication(NSAppleEventHandling) _handleAEOpenDocumentsForURLs:]_block_invoke + 917
	5   com.apple.AppKit              	0x00007fff8ad48a59 __97-[NSDocumentController(NSInternal) _autoreopenDocumentsIgnoringExpendable:withCompletionHandler:]_block_invoke_3 + 140
	6   com.apple.AppKit              	0x00007fff8ad485a1 -[NSDocumentController(NSInternal) _autoreopenDocumentsIgnoringExpendable:withCompletionHandler:] + 798
	7   com.apple.AppKit              	0x00007fff8ad46cc6 -[NSApplication _reopenWindowsAsNecessaryIncludingRestorableState:registeringAsReady:completionHandler:] + 331
	8   com.apple.AppKit              	0x00007fff8ae6e885 -[NSApplication(NSAppleEventHandling) _handleAEOpenDocumentsForURLs:] + 306
	9   com.apple.AppKit              	0x00007fff8ad46696 -[NSApplication(NSAppleEventHandling) _handleCoreEvent:withReplyEvent:] + 757
	10  com.apple.Foundation          	0x00007fff90f2a748 -[NSAppleEventManager dispatchRawAppleEvent:withRawReply:handlerRefCon:] + 290
	11  com.apple.Foundation          	0x00007fff90f2a5b9 _NSAppleEventManagerGenericHandler + 102
	12  com.apple.AE                  	0x00007fff92ec334c aeDispatchAppleEvent(AEDesc const*, AEDesc*, unsigned int, unsigned char*) + 531
	13  com.apple.AE                  	0x00007fff92ec30c9 dispatchEventAndSendReply(AEDesc const*, AEDesc*) + 31
	14  com.apple.AE                  	0x00007fff92ec2fd3 aeProcessAppleEvent + 295
	15  com.apple.HIToolbox           	0x00007fff8d1d4c6e AEProcessAppleEvent + 56
	16  com.apple.AppKit              	0x00007fff8ad3feb2 _DPSNextEvent + 2249
	17  com.apple.AppKit              	0x00007fff8ad3ef68 -[NSApplication nextEventMatchingMask:untilDate:inMode:dequeue:] + 346
	18  com.apple.AppKit              	0x00007fff8ad34bf3 -[NSApplication run] + 594
	19  libqcocoa.dylib               	0x0000000107c812fd 0x107c61000 + 131837
	20  org.qt-project.QtCore         	0x0000000105e5b37d QEventLoop::exec(QFlags<QEventLoop::ProcessEventsFlag>) + 381
	21  org.qt-project.QtCore         	0x0000000105e5e35a QCoreApplication::exec() + 346
	22  com.patgame.flashfire         	0x0000000104a5f7c8 cocos2d::CCApplication::run() + 120
	23  com.patgame.flashfire         	0x0000000104a9af56 main + 70
	24  com.patgame.flashfire         	0x00000001049a9b34 start + 52

一直调试，最后在[QT论坛上面找到答案](https://forum.qt.io/topic/18217/crash-on-launch-with-file-on-mac)。

我继承了QApplication，

	class CCApplication : public QApplication
	{
	    Q_OBJECT
	public:
	    CCApplication(int argc, char** argv) : QApplication(argc, argv)
	    {
	    }
	    
注意构造函数，那个argc是传值，应该修改成传引用:

	class CCApplication : public QApplication
	{
	    Q_OBJECT
	public:
	    CCApplication(int& argc, char** argv) : QApplication(argc, argv)
	    {
	    }
	    
QApplication的构造函数参数为

	QApplication(int &argc, char **argv);
	
假如CCApplication传值，QApplication就引用了一个临时数据。引用的本质就是指针，这样调用QCoreApplication::arguments()的时候，就使得argc的数据错误，之后再用argc的信息来访问argv，就引起了数组越界。

直接打开应用程序没有崩溃，是因为没有从来没有调用QCoreApplication::arguments，双击打开文件就触发了这个bug。