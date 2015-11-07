---
title: C++ 编程规范（4）-- 作用域
layout: post
published: true
---

作用域，表示某段代码或者数据的生效范围。作用域越大，修改代码时候影响区域也就越大，原则上，作用域越小越好。

4.1 全局变量
-----
禁止使用全局变量。全局变量在项目的任何地方都可以访问。两个看起来没有关系的函数，一旦访问了全局变量，就会产生无形的依赖。使用全局变量，基本上都是怕麻烦，贪图方便。比如

    funA -> funB -> funC -> funD

上图表示调用顺序。当funD需要用到funA中的某个数据。正确的方式，是将数据一层层往下传递。但因为这样做，需要修改几个地方，修改的人怕麻烦，直接定义出全局变量。这样做，当然是可以快速fix bug。但funA跟funD就引入无形的依赖，从接口处看不出来。

单件可以看做全局变量的变种。最优先的方式，应该将数据从接口中传递，其次封装单件，再次使用函数操作静态数据，最糟糕就是使用全局变量。

若真需要使用全局变量。变量使用g_开头。


<a name="member_variable"></a>
4.2 类的成员变量
-----
类的成员变量，只能够是private或者public, 不要设置成protected。protected的数据看似安全，实际只是一种错觉。

数据只能通过接口来修改访问，不要直接访问。这样的话，在接口中设置个断点就可以调试知道什么时候数据被修改。另外改变类的内部数据表示，也可以维持接口的不变，而不影响全局。

绝大多数情况，数据都应该设置成私有private, 变量加 _前缀。比如

	class Data
	{
	private:
		const uint8_t*  _bytes;
		size_t          _size;
	}

公有的数据，通常出现在C风格的结构中，或者一些数据比较简单，并很常用的类，public数据不要加前缀。

	class Point
	{
	public:
		Point(float x_, float y_) : x(x_), y(y_)
		{
		}

		.....

		float x;
		float y;
	}
注意，我们在构造函数，使用 x_ 的方式表示传入的参数，防止跟 x 来重名。


<a name="local_variable"></a>
4.3 局部变量
-----
局部变量尽可能使它的作用范围最小。换句话说，就是需要使用的时候才定义，而不要在函数开始就全部定义。

从前C语言有个约束，需要将用到的全部变量都定义在函数最前面。之后这个习惯也被传到C++的代码当中。但这种习惯是很不好的。

* 在函数最前面定义变量，变量就在整个函数都可见，作用域越大，就越容易被误修改。
* C++ 中，定义类型的变量，需要调用构造函数，跟释放函数。很多时候函数中途就退出了，这时候调用构造函数和释放函数，就显得浪费。
* 变量在最开始的时候，很难给变量一个合理的初始值，很难的话，也就很容易忘记。

我们的结论是，局部变量真正需要使用的时候才定义，一行定义一个变量，并且一开始就给它一个合适的初始值。

	int i;
	i = f();     // 错，初始化和定义分离
	int j = g(); // 对，定义时候给出始值


<a name="namespace"></a>
4.4 命名空间
-----
C++中，尽量不要出现全局函数，应该放入某个命名空间当中。命名空间将全局的作用域细分，可有效防止全局作用域的名字冲突。

比如

	namespace json
	{
		class Value
		{
			....
		}
	}

	namespace splite
	{
		class Value
		{
			...
		}
	}

两个命名空间都出现了Value类。外部访问时候，使用 json::Value, splite::Value来区分。

<a name="file_scrope"></a>
4.5 文件作用域
------
假如，某个函数，或者类型，只在某个.cpp中使用，请将函数或者类放入匿名命名空间。来防止文件中的函数导出。比如

	// fileA.cpp
	namespace
	{
		void doSomething()
		{
			....
		}
	}

上述例子，doSomething这个函数，放入了匿名空间。因此，此函数限制在fileA.cpp中使用。另外的文件定义相同名字的函数，也不会造成冲突。

另外传统C的做法，是在 doSomething 前面加 static, 比如

	// fileB.cpp
	static void doSomething()
	{
		...
	}

doSomething也限制到文件fileB.cpp中。

同理，只在文件中出现的类型，也放到匿名空间中。比如

	// sqlite/Value.cpp
	namespace sqlite
	{
		namespace
		{
			class Record
			{
				....
			}
		}
	}

上述例子，匿名空间嵌套到sqlite空间中。这样Record这个结构只可以在sqlite/Value.cpp中使用，就算是同属于空间sqlite的文件，也不知道 Record 的存在。

<a name="no_using_namespace"></a>
4.6 头文件不要出现 using namespace ....
------
头文件，很可能被多个文件包含。当某个头文件出现了 using namespace ... 的字样，所有包含这个头文件的文件，都简直看到此命令空间的全部内容，就有可能引起冲突。比如

	// Test.h
	#include <string>
	using namespace std;

	class Test
	{
	public:
		Test(const string& name);
	};

这个时候，只要包含了Test.h, 就都看到std的所有内容。正确的做法，是头文件中，将命令空间写全。将 string, 写成 std::string, 这里不要偷懒。
