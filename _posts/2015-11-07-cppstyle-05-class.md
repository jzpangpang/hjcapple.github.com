---
title: C++ 编程规范（5）-- 类
layout: post
published: true
---

面向对象编程中，类是基本的代码单元。本节列举了在写一个类的时候，需要注意的事情。

<a name="type_interface"></a>
## 5.1 让类的接口尽可能小
设计类的接口时，不要想着接口以后可能有用就先加上，而应该想着接口现在没有必要，就直接去掉。这里的接口，你可以当成类的成员函数。添加接口是很容易的，但是修改，去掉接口会会影响较大。

接口小，不单指成员函数的数量少，也指函数的作用域尽可能小。

比如，

	class Test
	{
	public:
		void funA();
		void funB();
		void funC();
		void funD();
	};

假如，funD 其实是可以使用 funA, funB, funC 来实现的。这个时候，funD，就不应该放到Test里面。可以将funD抽取出来。funD 只是一个封装函数，而不是最核心的。

	void Test_funD(Test* test);

编写类的函数时候，一些辅助函数，优先采用 Test_funD 这样的方式，将其放到.cpp中，使用匿名空间保护起来，外界就就不用知道此函数的存在，那些都只是实现细节。

当不能抽取独立于类的辅助函数，先将函数，变成private, 有必要再慢慢将其提出到public。 不要觉得这函数可能有用，一下子就写上一堆共有接口。

再强调一次，如无必要，不要加接口。

从作用域大小，来看

* 独立于类的函数，比类的成员函数要好
* 私有函数，比共有函数要好
* 非虚函数，比虚函数要好

<a name="order"></a>
## 5.2 声明顺序
类的成员函数或者成员变量，按照使用的重要程度，从高到低来排列。

比如，使用类的时候，用户更关注函数，而不是数据，所以成员函数应该放到成员变量之前。
再比如，使用类的时候，用户更关注共有函数，而不是私有函数，所以public，应该放在private前面。

具体规范

* 按照 public, protected, private 的顺序分块。那一块没有，就直接忽略。

每一块中，按照下面顺序排列

* typedef，enum，struct，class 定义的嵌套类型
* 常量
* 构造函数
* 析构函数
* 成员函数,含静态成员函数
* 数据成员,含静态数据成员

.cpp 文件中，函数的实现尽可能给声明次序一致。

<a name="inheritance"></a>
## 5.3 继承
优先使用组合，而不是继承。

继承主要用于两种场合：实现继承，子类继承了父类的实现代码。接口继承，子类仅仅继承父类的方法名称。

我们不提倡实现继承，实现继承的代码分散在子类跟父亲当中，理解起来变得很困难。通常实现继承都可以采用组合来替代。

规则：

* 继承应该都是 public
* 假如父类有虚函数，父类的析构函数为 virtual
* 假如子类覆写了父类的虚函数，应该显式写上 override

比如

	// swf/Definition.h
	class Definition
	{
	public:
		virtual ~Definition()	{}
		virtual void parse(const uint8_t* bytes, size_t len) = 0;
	};

	// swf/ShapeDefinition.h
	class ShapeDefinition : public Definition
	{
	public:
		ShapeDefinition()	{}
		virtual void parse(const uint8_t* bytes, size_t len) override;

	private:
		Shape	_shape;
	};

<br>

	Definition* p = new ShapeDefinition();
	....
	delete p;

上面的例子，使用父类的指针指向子类，假如父类的析构函数不为virtual, 就只会调用父类的Definition的释放函数，引起子类独有的数据不能释放。所有需要加上virtual。

另外子类覆写的虚函数写上，override的时候，当父类修改了虚函数的名字，就会编译错误。从而防止，父类修改了虚函数接口，而忘记修改子类相应虚函数接口的情况。