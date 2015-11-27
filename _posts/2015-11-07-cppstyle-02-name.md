---
title: C++ 编程规范（2）-- 命名约定
layout: post
published: true
---

## 2.1 使用英文单词，不能夹着拼音
这条规则强制执行，不能有例外。

<a name="camel"></a>
## 2.2 总体上采用骆驼命名法
单词与单词之间，使用大小写相隔的方式分开，中间不包含下划线。比如

	TimerManager  // (1)
	playMusic     // (2)

其中(1)为大写的骆驼命名法，(2)为小写的骆驼命名法。

不要使用

	timer_manager
	play_music

这种小写加下划线的方式，这种加下划线的方式在boost库，C++标准库中，用得很普遍。

接下来分别描述具体的命名方式。

<a name="can_not_type_prefix"></a>
## 2.3 名字不要加类型前缀
有些代码库，会在变量名字前面加上类型前缀。比如 b表示bool, i表示int, arr表示数组, sz表示字符串等等。他们会命名为

    bool          bEmpty;
    const char*   szName;
    Array         arrTeachers;

我们不提倡这种做法。变量名字应该关注用途，而不是它的类型。上面名字应该修改为

    bool        isEmpty;
    const char* name;
    Array       teachers;

注意，我们将 bool 类型添加上is。isEmpty, isOK, isDoorOpened，等等，读起来就是一个询问句。

<a name="type_name"></a>
## 2.4 类型命名
类型命名采用大写的骆驼命名法，每个单词以大写字母开头，不包含下划线。比如

	GameObject
	TextureSheet

类型的名字，应该带有描述性，是名词，而不要是动词。尽量避开Data, Info, Manager 这类的比较模糊的字眼。(但我知道有时也真的避免不了，看着办。)

所有的类型，class, struct, typedef, enum, 都使用相同的约定。例如

	class UrlTable
	struct UrlTableProperties
	typedef hash_map<UrlTableProperties*, std::string> PropertiesMap;
	enum UrlTableError

<a name="variable_name"></a>
## 2.5 变量命名
### 2.5.1 普通变量名字
变量名字采用小写的骆驼命名法。比如

	std::string tableName;
	CCRect      shapeBounds;

变量的名字，假如作用域越长，就越要描述详细。作用域越短，适当简短一点。比如

	for (auto& name : _studentNames)
	{
        std::cout << name << std::endl;
	}

	for (size_t i = 0; i < arraySize; i++)
	{
		array[i] = 1.0;
	}

名字清晰，并且尽可能简短。

### 2.5.2 类成员变量
成员变量，访问权限只分成两级，private 和 public，不要用 protected。
私有的成员变量，前面加下划线。比如：

	class Image
	{
	public:
		.....

	private:
		size_t    _width;
		size_t    _height;
	}

public 的成员变量，通常会出现在C风格的struct中，前面不用加下划线。比如：

    struct Color4f
    {
        float    red;
        float    green;
        float    blue;
        float    alpha;
    }

### 2.5.3 静态变量
类中尽量不要出现静态变量。类中的静态变量不用加任何前缀。文件中的静态变量统一加s_前缀，并尽可能的详细命名。比如

    static ColorTransformStack s_colorTransformStack;    // 对
    static ColorTransformStack s_stack;                  // 错（太简略）

### 2.5.4 全局变量
不要使用全局变量。真的没有办法，加上前缀 g_，并尽可能的详细命名。比如

    Document  g_currentDocument;

<a name="function_name"></a>
## 2.6 函数命名
变量名字采用小写的骆驼命名法。比如

    playMusic
    getSize
    isEmpty

函数名字。整体上，应该是个动词，或者是形容词(返回bool的函数)，但不要是名词。

    teacherNames();        // 错（这个是总体是名词）
    getTeacherNames();     // 对

无论是全局函数，静态函数，私有的成员函数，都不强制加前缀。但有时静态函数，可以适当加s_前缀。

类的成员函数，假如类名已经出现了某种信息，就不用重复写了。比如

    class UserQueue
    {
    public:
        size_t getQueueSize();    // 错(类名已经为Queue了，
                                  // 这里再命名为getQueueSize就无意义)
        size_t getSize();         // 对
    }

<a name="namespace"></a>
## 2.7 命名空间
命令空间的名字，使用小写加下划线的形式，比如

    namespace lua_wrapper;

使用小写加下划线，而不要使用骆驼命名法。可以方便跟类型名字区分开来。比如

    lua_wrapper::getField();  // getField是命令空间lua_wrapper的函数
    LuaWrapper::getField();   // getField是类型LuaWrapper的静态函数

<a name="macro_name"></a>
## 2.8 宏命名
不建议使用宏，但真的需要使用。宏的名字，全部大写，中间加下划线相连接。这样可以让宏更显眼一些。比如

    #define PI_ROUNDED 3.0
    CLOVER_TEST
    MAX
    MIN

头文件出现的防御宏定义，也全部大写，比如：

    #ifndef __COCOS2D_FLASDK_H__
    #define __COCOS2D_FLASDK_H__

    ....

    #endif

不要写成这样：

    #ifndef __cocos2d_flashsdk_h__
    #define __cocos2d_flashsdk_h__

    ....

    #endif



<a name="enum_name"></a>
## 2.9 枚举命名
尽量使用 0x11 风格 enum，例如：

    enum class ColorType : uint8_t
    {
        Black,
        While,
        Red,
    }

枚举里面的数值，全部采用大写的骆驼命名法。使用的时候，就为 ColorType::Black

有些时候，需要使用0x11之前的enum风格，这种情况下，每个枚举值，都需要带上类型信息，用下划线分割。比如

    enum HttpResult
    {
        HttpResult_OK     = 0,
        HttpResult_Error  = 1,
        HttpResult_Cancel = 2,
    }

<a name="c_like"></a>
## 2.10 纯 C 风格的接口
假如我们需要结构里面的内存布局精确可控，有可能需要编写一些纯C风格的结构和接口。这个时候，接口前面应该带有模块或者结构的名字，中间用下划线分割。比如

    struct HSBColor
    {
        float h;
        float s;
        float b;
    };

    struct RGBColor
    {
        float r;
        float g;
        float b;
    }

    RGBColor color_hsbToRgb(HSBColor hsb);
    HSBColor color_rgbToHsb(RGBColor rgb);

这里，color 就是模块的名字。这里的模块，充当C++中命名空间的作用。

    struct Path
    {
        ....
    }

    Path* Path_new();
    void  Path_destrory(Path* path);
    void  Path_moveTo(Path* path, float x, float y);
    void  Path_lineTo(Path* path, float x, float y);

这里，接口中Path出现的是类的名字。

<a name="file_path_name"></a>
## 2.11 代码文件，路径命名
代码文件的名字，应该反应出此代码单元的作用。

比如 Point.h, Point.cpp，实现了class Point;

当 class Point，的名字修改成，Point2d, 代码文件名字，就应该修改成 Point2d.h, Point2d.cpp。代码文件名字，跟类型名字一样，采用大写的骆驼命名法。

路径名字，对于于模块的名字。跟上一章的命名规范一样，采用小写加下划线的形式。比如

    ui/home/HomeLayer.h
    ui/battle/BattleCell.h
    support/geo/Point.h
    support/easy_lua/Call.h

路径以及代码文件名，不能出现空格，中文，不能夹着拼音。假如随着代码的修改，引起模块名，类型名字的变化，应该同时修改文件名跟路径名。


<a name="can_not_personal_name"></a>
## 2.12 命名避免带有个人标签
比如，不要将某个模块名字为

    HJCPoint
    hjc/Label.h

hjc为团队某人名字的缩写。

项目归全体成员所有，任何人都有权利跟义务整理修改工程代码。当某样东西打上个人标记，就倾向将其作为私有。其他人就会觉得那代码乱不关自己事情，自己就不情愿别人来动自己东西。

当然了，文件开始注释可以出现创建者的名字，信息。只是类型，模块，函数名字，等容易在工程中散开的东西不提倡。个人项目可以忽略这条。

再强调一下，任何人都有权利跟义务整理修改他人代码，只要你觉得你修改得合理，但不要自作聪明。我知道有些程序员，会觉得他人修改自己代码，就是入侵自己领土。

<a name="exception"></a>
## 2.13 例外
有些时候，我们需要自己写的库跟C++的标准库结合。这时候可以采用跟C++标准库相类似的风格。比如

    class MyArray
    {
    public:
        typedef const char* const_iteator;
        ...

        const char* begin() const;
        const char* rbegin() const;
    }
