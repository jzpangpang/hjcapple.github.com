---
title: OS X 静态编译 Qt 5.4.1
layout: post
published: true

---
在官网下载离线安装包。安装的时候选择包含代码。

假设Qt安装在home目录。

打开终端

    cd ~/Qt5.4.1/5.4/Src
    ./configure -prefix $PWD/qtbase -opensource -nomake tests -static
    
-static选项就是为了配置静态编译。配置之后，产生Makefile文件。再敲入以下命令进行编译。

    make -j4
    
根据机子的性能编译时间有所不同，十几分钟后，编译完成。在 ~/Qt5.4.1/5.4/Src/qtbase 下可以找到编译出来的静态库。

| qtbase下目录|作用
|------------|------------------:
| include |  包含对应的头文件
| lib | 编译出的主要静态库
| plugins | 编译出的静态插件
 
使用方式跟平时的静态库一样。设置头文件路径，设置库路径，选择对应的库。

一个GUI程序。需要链接的Qt库库大致为

* libQt5Core.a
* libQt5Gui.a
* libQt5Widgets.a
* libQt5PlatformSupport.a
* libQt5PrintSupport.a
* libqcocoa.a
* libqthartbuzzng.a

其中libqcocoa.a在 qtbase/plugins/platforms 下

OS X的系统库，包括

* libcups.dylib
* libz.dylib
* Cocoa.framework
* Carbon.framework
* OpenGL.framework
* IOKit.framework

另外静态编译的时候。需要在main.cpp文件的main函数前面加上静态注册cocoa插件的代码。例如

    #include "mainwindow.h"
    #include <QtWidgets/QApplication>
    #include <QtCore/QtPlugin>
    
    Q_IMPORT_PLUGIN(QCocoaIntegrationPlugin)
    
    int main(int argc, char *argv[])
    {
        QApplication a(argc, argv);
        MainWindow w;
        w.show();
        return a.exec();
    }
    
之后编译出来的程序就可以单独运行，不用包含对应的动态库。


