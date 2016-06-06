---
title: ScopeGuard 介绍和实现
layout: post
published: true
---

## 介绍

程序库，可粗分成两类。一类解决具体特定问题，比如 json 解释, base64 编码，访问数据库，实现某个动画效果。另一类库辅助表达的，用来更方便地写代码，比如 map, filter，reduce 等函数，RxSwift，BlocksKit 等库。第二类库的抽象程度往往更高，需要掌握某种思想，甚至可以改变你某种观念。

在这里，我介绍一个很基本但很重要的库，叫 ScopeGuard。实际上说库也不算很对，应该算是某种观念。ScopeGuard 是在 C++ 中的叫法，我不清楚类似的概念在其它场合叫做什么。

这个东西有无数人说了无数次，都被说烂了。只是我悲哀地发现，还是有无数人不知道，前赴后继地产生坏代码。

几乎所有的编程语言，都有作用域（Scope）的概念，变量出了作用域就不能被外面访问了。说起来抽象，举个例子就很简单了。比如下面 Objective-C 代码：

``` Objective-C
int a = 1;
{
    int b = 1;
    xxxx
    // (1)
}
// (2)
```

Objective-C 中使用大括号就形成作用域（有些语言用 begin, end 形成作用域）。在 (1) 中可以访问 a 变量，而出了作用域之后（2）中不能访问 b 变量。这个概念看起来似乎是无需解释，甚至是理所当然的。但 JavaScript 这种语言就比较奇葩，只有函数作用域，大括号括起来的不算，这样就是一个大坑，很容易犯错。

ScopeGuard 就是出作用域后，自动执行某段代码。比如如下 Swift 代码：

``` Swift
{
    defer {
        print("print 2");
    }
    print("print 1");
}
```

defer 实质上就是 ScopeGuard，定义的时候里面的代码还不会被执行。而一旦出了作用域，里面的代码就会被执行，所以最后就输出:

```
print 1
print 2
```

在不同的环境当中，ScopeGuard 的叫法可能不同。在 Swift 中用 defer 这个词，是延迟执行的意思。另外的场合，可以叫 onExit, ON_SCOPE_EXIT。重要的是它的含义，而不是它的名字。

来到这里，只描述了 ScopeGuard 的语法和表象。接下来的问题就是为什么要有这个东西，这个东西有什么用处？不知道它的用处，记再多的语法细节都没用的。

ScopeGuard 最大的用处就是释放资源。

比如打开文件，做某些操作，再关闭文件，很多人会这样写：

``` C
FILE* file = fopen(filename, "rb");
xxxxx
if (xxx)
{
    xxxxx
}
else
{
    xxxxx
}

fclose(file);
```

这样的代码是很脆弱的，fopen 和 fclose 相隔太远，难以看出它们的对应关系。另外中间查产生有任何异常，中途 return，fclose 都不能被执行，就有资源泄露。这种代码实现起来要很小心，修改起来要很小心。假如某段代码要很小心，就会有问题。可能会犯错的就一定会犯错，再小心的人也有犯糊涂的时候。

为解决这种资源释放问题，有些老旧的 C/C++ 代码，甚至会用到 goto。比如：

``` C
    if (xxxx) {
        goto fail;
    }

    if (xxxx) {
        goto fail;
    }

    xxx 正常操作
    return 1;

fail:
    fclose(file);
    xxxx 释放资源
    return 0;
```

而某系团队为了避免中途 return 而忘记释放资源，甚至还会设定规则：一个函数只能有出现一个 return，不能中途 return。这种规则随着时间推移，到最后无人知道为什么，还一直被当做金科玉律地固守着。一个函数只能一个 return, 往往会引入多余的嵌套，而难以读懂。

有了 ScopeGuard，上述问题解决起来就很简单，可以这样写代码：

``` C
FILE* file = fopen(filename, "rb");
defer {
    fclose(file);
};

xxxxx
if (xxx)
{
    xxxxx
}
else
{
    xxxxx
}
```

资源一旦分配，接下来就立即使用 ScopeGuard 释放资源，这样分配和释放就会靠在一起，当退出作用域的的时候，里面的语句就被执行，释放掉资源。无论下面的语句抛异常也好，中途退出也好，代码都是安全的。这样的代码也更容易修改，比如将其移动到另外的地方，它还是安全的。假如分配和释放分隔两地，移动代码时就很容易漏掉某些语句。

有时并非用于释放资源，而仅仅为了代码匹配，也可以使用 ScopeGuard。比如：

```
CGContextSaveGState(context);
defer {
    CGContextRestoreGState(context);
};

drawer.beginDraw(); 
defer {
    drawer.endDraw();
}
```

实际上这种代码匹配，也可看做是广义上的资源释放。ScopeGuard 这个简单有效的基本设施，会改变你编写代码的方式。每切换一种语言，我都会看看它有没有在语言中内置，假如没有，就想办法动手实现一个。没有这个小东西，编写安全的代码就很麻烦了，总会在某些细节上纠缠。

## 实现

Swift、Go、D 语言中是内置的，关键字是 defer。C++ 和 Objective-C 语言中，没有对应的东西，但我们可以用库来实现。

为示范，我就实现了一个，放到 [GitHub](https://github.com/hjcapple/ScopeGuard) 上。使用如下：

``` C++
static void test_cpp()
{
    void* p = malloc(1000);
    ON_SCOPE_EXIT
    {
        free(p);
    };
}
```

这个 ON_SCOPE_EXIT 宏，作用就跟 Swift 的 defer 关键字一样。可以在 C++ 和 Objective-C 中（当然也可以用于 C 语言当中）当中使用。原则上，C++/Objective-C 是不应该使用宏的，只是有些情况，不用宏还真实现不了。通常习惯是，宏都用大写，让它丑陋一点。

最后说一说，这个实现的一些小技巧。

先使用 #if defined(__cplusplus) 区分是 C++ 代码，还是 Objective-C 代码，再分别实现。

C++ 实现用到 C++ 11 的 lamda，定义对象将 lamda 存起来，在析构函数中调用。这个 lamda 在不同的场合也有不同的叫法，比如匿名函数，闭包，代码块，block。叫什么名字根本不重要，就算叫其它，它还是那种东西。我很少记东西，也经常忘记名字，需要查查才知道。

C++ 实现有些小地方需要注意：

一个是存储 lamda 使用了模板，而不是 std::function, 这个可以避免 lamda 转 std::function 的开销（尽管这个开销在绝大多数情况下可以忽略不计）。

另外一个小细节是实现了一个 operator+ 的辅助函数，这是想将构造函数的一些多余括号去掉。展开大致为：

```
auto ext_exitBlock_11 = clover::detail::ScopeGuardOnExit() + [&] {
};
```

这样将前面的语句转成宏，就可以写成

```
ON_SCOPE_EXIT
{
    xxx
};
```

网上搜索到大多数实现，使用了辅助函数 MakeScopeGuard，它的展开大致为

```
auto ext_exitBlock_11 = MakeScopeGuard([&] {
});
```

这样就会出现多余的大括号，实现的宏就变得比较丑陋了。

```
ON_SCOPE_EXIT([&] {
});
```

而 Objective-C 的实现（实质也是 C 中的实现）需要用到 gcc 的 `__attribute__((cleanup))` 扩展，这个扩展在变量退出作用域时，可以指定执行某个函数, 在 Objective-C 中，ON_SCOPE_EXIT 展开为

```
typedef void (^ext_cleanupBlock_t)();
__strong ext_cleanupBlock_t ext_exitBlock_11  __attribute__((cleanup(ext_executeCleanupBlock), unused)) = ^{
};
```

定义出一个 block 变量，之后变量退出的时候执行 ext_executeCleanupBlock，在其中调用 block。这样也就执行了其中的语句，封装成宏，跟 C++ 的实现使用一致。

```
ON_SCOPE_EXIT {
};
```

这个实现参考了 [libextobjc](https://github.com/jspahrsummers/libextobjc) 库。

代码中还包含了一个 CFScopeRelease.h 的文件，实现了 CF_SCOPE_RELEASE 宏。在 Objective-C 中，经常创建一些 CFTypeRef, 需要 Release 掉。比如创建了 CGContextRef, 就需要调用 CGContextRelease，这种 Release 的操作实在太频繁，就算使用 ON_SCOPE_EXIT 也嫌麻烦，就特定定义了宏 CF_SCOPE_RELEASE

就可以写成

```
CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
CF_SCOPE_RELEASE(colorSpace);

CGContextRef context = CreateBitmapRGBA8Context(bytes, width, height);
CF_SCOPE_RELEASE(context);
```

创建出来的对象，再下一行就先清理掉，这个原理跟 ScopeGuard 是一样的，实现起来也差不多。

这样写代码，就没有后患。

我希望这种基础设施，可以集成到语言或者标准库当中，让更多的人知道，可以少些错。