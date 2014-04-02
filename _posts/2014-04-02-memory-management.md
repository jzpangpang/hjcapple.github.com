---
title: 内存管理
layout: post
published: false

---

最近遇到个循环引用的问题，最后用了弱指针来解决。就打算写个弱指针相关的文章，后发觉要想好好讲解弱指针，就需要先讲解引用计数。而要讲解引用计数，牵涉到内存管理。

因此还是从内存管理讲解起。

写代码的时候，应职责分离，自己管好自己。当写一个函数的时候，这里自己就是指函数。写一个类的时候，自己就指类，写一个模块的时候，自己就指模块。函数管好自己，类管好自己，模块管好自己。这样就可以让函数，类，模块尽可能独立，也方便测试和调试。

对于内存，自己管好自己，就是谁分配，谁就负责释放。



1. cocoa的对象，内存基本上分配在堆上面。id，NSObject* 相当于指针，指向这个对象。这样产生怎么释放的问题，需要一定的内存管理策略。

2. cocoa框架的内存管理使用了引用计数。retain增加引用计数，release减少引用计数。当引用计数为0的时候，会回收内存。这种机制足够简单而有效。

3. 写代码时候，应职责分离，自己管好自己。对于内存，具体说，就是谁分配，谁负责释放。这样就会产生一个问题。
NSString* getTitle(); 这个函数返回的对象到底是外面调用者释放，还是getTitle函数本身释放。
1) 如果是外面释放，就违反了谁分配的，谁负责释放的原则。让外面释放会很容易忘记，产生泄露。
2) 如果是getTitle里面释放，返回外面的就是野指针，因为对象已经不存在了。
这是一个矛盾，这个问题很多人问。也很多人不理解。

4. 先看看C和C++解决类似问题的方法。
1) 有些C函数会出现这样的接口 int getTitle(const char* buf, int len), 要求外面先分配内存，之后在函数里面填充数据，之后外面再释放。应该分配多大内存呢？就先用null指针调用，getTitle(NULL, 0)会返回需要的内存大小。这样没有违背原则，但需要两次调用。

2) C++做法，会将原始指针用类封装起来，返回一个智能指针。StringPtr getTitle(), 智能指针释构函数会自动调用，就会自动将内存释放。C++也经常用引用计数，但会将引用计数用智能指针封装起来，很少看到手动调用retain, release的情况。

5. object-c的解决方法。
为解决这个矛盾，object-c在引用计数的基础上，加上autorelease。
每个线程入口，会有个NSAutoreleasePool，调用对象的autorelease,  就会将此对象加到pool中，相当于有一个引用计数被NSAutoreleasePool管理。
主函数的入口在main.m中，看到有个NSAutoreleasePool，或者@autorelease（只是种简写）。如果是我们自己创建一个新线程，也需要在线程入口处生成一个NSAutoreleasePool

NSString* getTitle()
{
    return [[NSString alloc] initWithString:@"hello"] autorelease];
}
函数里面将对象加入自动释放池，外面调用完这个函数的时候，对象的引用计数为1（其中一个被pool管理），这个时候对象还没有被释放，外面就可以使用，也不用外面来释放。

6. 那放在NSAutoreleasePool的对象什么时候被释放呢？
主线程有个事件循环。每次有事件的时候，由cocoa框架调用我们写的代码，再回到cocoa框架。NSAutoreleasePool其实是由cocoa框架管理的，当出了我们写的代码，回到框架的时候，放入NSAutoreleasePool的对象就会由框架调用release, 这样就保证跑我们代码的时候，对象是会存在的，出了我们的代码，对象也能够正常地被释放。

比如
系统代码（事件激发）
funA()
funB()
funC()
funD()
系统代码（等待事件）

在funA, funB, funC, funD中，无论调用多少层，也是用户写的代码。autorelease的对象还是存在的。下一次事件激发的时候，上一次事件的对象是不存在的，因为已经被系统框架调用release回收了。除非你自己调用了retain, 自己调用了一次retain, 就应该自己来调用一次release, 这样也没有违背上面说的原则。

7. 另外NSAutoreleasePool其实是可以嵌套的。
NSAutoreleasePool* poolA = [[NSAutoreleasePool alloc] init];
NSString* title = [[NSString alloc] initWithString:@"hello"] autorelease];
[poolA release];
定义了poolA之后，最上层的pool是poolA, title就会加到poolA中，poolA释放掉，加到poolA的对象也会被调用release，这样title就被释放了。


8. cocoa框架的约定。每个类分配对象的时候，多数会有两个版本的函数。比如NSString的
- initWithString，注意是减号，表示对象函数。用法 [[NSString alloc] initWithString:@"hello"], 这种init开头的函数，需要一次release。
+ stringWithString, 注意是加号，表示类函数。用法 [NSString strignWithString:@"helo"], 这种stringWith开头的函数，里面已经将返回对象放到autorelase,不用调用release. 
如果是 NSArray, 会有类似函数 + arrayWithXXXX, NSDictioanry，会有函数+ dictWithXXX。NSSet, 会有setWithXXX. 等等。这种是约定的规范。自己的代码也应该按照这种规范。


