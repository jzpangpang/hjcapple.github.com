---
title: C++ 编程规范（3）-- 代码文件
layout: post
published: true
---

## 3.1 \#define 保护
所有的头文件，都应该使用#define来防止头文件被重复包含。命名的格式为

    __<模块>_<文件名>_H__

很多时候，模块名字都跟命名空间对应。比如

    #ifndef __GEO_POINT_H__
    #define __GEO_POINT_H__

    namespace geo
    {
        class Point
        {
            .....
        };
    }

    #endif

并且，#define宏，的名字全部都为大写。不要出现大小写混杂的形式。

<a name="include_order"></a>
## 3.2 \#include 的顺序
C++代码使用#include来引入其它的模块的头文件。尽可能，按照模块的稳定性顺序来排列#include的顺序。按照稳定性从高到低排列。

比如

    #include <map>
    #include <vector>
    #include <boost/noncopyable.hpp>
    #include "cocos2d.h"
    #include "json.h"
    #include "FlaSDK.h"
    #include "support/TimeUtils.h"
    #include "Test.h"

上面例子中。#include的顺序，分别是C++标准库，boost库，第三方库，我们自己写的跟工程无关的库，工程中比较基础的库，应用层面的文件。

但有一个例外，就是 .cpp中，对应的.h文件放在第一位。比如geo模块中的, Point.h 跟 Point.cpp文件，Point.cpp中的包含

    #include "geo/Point.h"
    #include <cmath>

这里，将 #include "geo/Point.h"，放到第一位，之后按照上述原则来排列#include顺序。理由下一条规范来描述。

<a name="header_file_dependence"></a>
## 3.3 尽可能减少头文件的依赖
代码文件中，每出现一次#include包含, 就会多一层依赖。比如，有A，B类型，各自有对应的.h文件和.cpp文件。

当A.cpp包含了A.h, A.cpp就依赖了A.h，我们表示为

    A.cpp -> A.h

这样，当A.h被修改的时候，A.cpp就需要重修编译。
假设

    B.cpp -> B.h
    B.h   -> A.h

这表示，B.cpp 包含了B.h, B.h包含了A.h, 这个时候。B.cpp虽然没有直接包含A.h, 但也间接依赖于A.h。当A.h修改了，B.cpp也需要重修编译。

当在头文件中，出现不必要的包含，就会生成不必要的依赖，引起连锁反应，使得编译时间大大被拉长。

使用前置声明，而不是直接#include，可以显著地减少依赖数量。实践方法：

### 3.3.1 头文件第一位包含

比如写类A，有文件 A.h, 和A.cpp
那么在A.cpp中，将A.h的包含写在第一位。在A.cpp中写成

    // 前面没有别的头文件包含
    #include "A.h"
    #include <string>
    #include .......
.... 包含其它头文件

之后可以尝试在 A.h 中去掉多余的头文件。当A.cpp可以顺利编译通过的时候，A.h包含的头文件就是过多或者刚刚好的。而不会是包含不够的。

### 3.3.2 前置声明
首先，只在头文件中使用引用或者指针，而不是使用值的，可以前置声明。而不是直接包含它的头文件。
比如

    class Test : public Base
    {
    public:
        void funA(const A& a);
        void funB(const B* b);
        void funC(const space::C& c);

    private:
        D   _d;
    };

这里，我牵涉到几个其它类，Base, A, B, space::C（C 在命名空间space里面）, D。Base和D需要知道值，A, B, space::C只是引用和指针。所以Base, C的头文件需要包含。A, B，space::C只需要前置声明。

    #include "Base.h"
    #include "D.h"

    namespace space
    {
        class C;
    }

    class A;
    class B;
    class Test : public Base
    {
    public:
        void funA(const A& a);
        void funB(const B* b);
        void funC(const space::C& c);

    private:
        D   _d;
    };

注意命名空间里面的写法。

### 3.3.3 impl 手法
就是类里面包含实现类的指针。在cpp里面实现。


<a name="break_down"></a>
### 3.3.4 尽可能将代码拆分成相对独立的，粒度小的单元，放到不同的文件中
简单说，就是不要将所有东西都塞在一起。这样的代码组积相对清晰。头文件包含也相对较少。但现实中，或多或少会违反。

比如，工程用到一些常量字符串（或者消息定义，或者enum值，有多个变种）。一个似乎清晰的结构，是将字符串都放到同一个头文件中。不过这样一来，这个字符串文件，就几乎会被所有项目文件包含。当以后新加一个字符串时候，就算只加一行，工程几乎被全部编译。

更好的做法，是按照字符串的用途来分拆开。

又比如，有些支持库。有时贪图方便，不注意的，就会写一个 GlobalUtils.h 之类的头文件，包含所有支持库，因为这样可以不关心到底应该包含哪个，反正包含GlobalUtils.h就行，这样多省事。不过这样一来，需要加一个支持的函数，比如就只是角度转弧度的小函数，也会发生连锁编译。

更好的做法，是根据需要来包含必要的文件。就算你麻烦一点，写10行#include的代码，都比之后修改一行代码，就编译上10多分钟要好。

## 3.4 小结
减少编译时间，这点很重要。再啰嗦一下

* 要减少头文件重复包含，需要团队的人所有人达成共识，认识到这是不好的。很多人对这问题认识不够，会被当成小题大作。
* 不要贪方便。直接包含一个大的头文件，短期是很方便，长期会有麻烦。


<a name="include_path"></a>
## 3.5 \#include中的头文件，尽量使用全路径，或者相对路径
-----
路径的起始点，为工程文件代码文件的根目录。

比如

    #include "ui/home/HomeLayer.h"
    #include "ui/home/HomeCell.h"
    #include "support/MathUtils.h"

不要直接包含

    #include "HomeLayer.h"
    #include "HomeCell.h"
    #include "MathUtils.h"

这样可以防止头文件重名，比如一个第三方库文件有可能就叫 MathUtils.h。

并且移植到其它平台，配置起来会更容易。比如上述例子，在安卓平台上，就需要配置包含路径

    <Project_Root>/ui/home/
    <Project_Root>/support/

也可以使用相对路径。比如

    #include "../MathUtil.h"
    #include "./home/HomeCell.h"

这样做，还有个好处。就是只用一个简单脚本，或者一些简单工具。就可以分析出头文件的包含关系图，然后就很容易看出循环依赖。