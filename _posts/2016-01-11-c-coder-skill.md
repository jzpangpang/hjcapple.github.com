---
title: c_coder 中用到的一些 C++ 小技巧
layout: post
published: true
---

[c_coder](https://github.com/hjcapple/c_coder) 是我编写的一个小小的库。使用 C++ 11 来输出简单的 C 代码。关于 c_coder 的来由用法可以参考文章：[使用 C++ 输出 C 代码](/2016/01/09/c-coder.html)

下面介绍一下编写 c_coder 时用到的一些 C++ 代码技巧。

## 链式接口
c_coder 定义了一种 DSL, 使用链式接口，风格跟传统的命令风格不一样。下面的例子就是链式接口。

	C_Coder coder;
	coder
	.begin()
	.call("sin")(30)
	.end();
	
传统的接口会写成。

	C_Coder coder;
	coder.begin();
	coder.call("sin", 30);
	coder.end();
	
为实现这种风格，可以每个函数返回自身的引用。比如：

    C_Coder& C_Coder::begin()
    {
        _buf += "{";
        br();
        indent();
        return *this;
    }
    
也可以返回一个对象，对象调用之后再返回另一对象。比如

	A -> B -> A&
	
调用 A 对象的函数返回 B 对象，B 对象构造时保存 A 的引用，之后调用 B 对象的函数再次返回 A 对象的引用。这样可以不断链式调用下去。
    
这种链式接口在 C++ 中其实也挺常见，只是之前没有意识到。比如

	std::cout << "Hello, World" << std::endl;
	
重载了运算符 <<，也是返回了自身引用。
	
这种风格并非 C++ 特有的，其它语言也可以定义这样的风格。比如 Objective-C 的 Masonry 库：
	
	make.width.lessThanOrEqualTo(self);
	make.height.lessThanOrEqualTo(self);
	
比如 Ruby on rails:

	3.days.ago
	
## 禁止外部构造对象和复制对象

	coder.call("sin")(100);
	
C_Coder 的 call 返回了一个 Caller 对象。而 Caller 又重载了运算符()

	class Caller : boost::noncopyable
	{
	public:
	    template <typename... Args>
	    C_Coder& operator()(Args&&... args)
	    xxxx
	}
	
	Caller call(const std::string& name)
	{
	    str(name);
	    return Caller(*this);
	}
	
为了防止用户误用，下面的代码应该直接编译失败。

	C_Coder coder;
	C_Coder::Caller caller;	                    // 错误，用户在外部创建了对象。
	C_Coder::Caller caler = coder.call("sin"); // 错误，用户在外部复制了对象。
	
防止用户创建对象，可以将构造函数设置为私有。禁止复制赋值，可以继承 boost::noncopyable。Caller 应该定义为：

	class Caller : boost::noncopyable
	{
	private:
		friend class C_Coder;
		Caller(Caller&& rhs) : _coder(rhs._coder)
		{
		}
		
	    Caller(C_Coder& coder) : _coder(coder)
	    {
	    }    
	};
	
`friend class C_Coder` 和 move 构造函数，让 C_Coder 内部可以调用`return Caller(*this);`。但外部是没有办法创建和保存 Caller 的。
	
接口应该保证容易用，并不容易被误用。无论多么小心，容易犯错的总有一天会犯错，就算怎么强调、说明、提醒、注意都没有用。

有些错误一旦出现就比较难查了，尽可能在还清醒的时候杜绝以后犯错的机会。不是不能犯错，只是不能犯一些低级水平的错。很多天灾本身就是人祸，很多麻烦是自找的。只是事前提醒无人会注意，而事后再提就是落井下石。

禁止构造或者禁止复制是很有用的。比如

	Data getFileData();
	
当数据很多时，Data 复制的代价就很高，这样可以直接禁止复制，防止一不小心就复制了。再提供一个 move 构造函数，让 Data 在函数内部创建。

## 将对象转成字符串
这个用了 boost::format 库，

	auto tmp = boost::format("%1%") % val;
	std::string str = tmp.str();
	
这段代码不管 val 为什么类型，只要支持流输出。就可以转成字符串。

## 不固定参数个数
	
	coder.call("doIt")(100);
	coder.call("doIt")(100, 100);
	
Caller 重载了符号(), 这个函数需要处理不固定的参数个数。这个需求在 C++ 11 之前只能定义多个函数。
	
	template <typename T0>
	C_Coder& operator()(T0&& arg0);
	
	template <typename T0, typename T1>
	C_Coder& operator()(T0&& arg0, T1&& arg1);
	
	template <typename T0, typename T1, typename T2>
	C_Coder& operator()(T0&& arg0, T0&& arg1, T2&& arg1);

在 C++ 11 中出现不定参数模板，可以将接口统一写成。

	template <typename... Args>
	C_Coder& operator()(Args&&... args)
	
可以定义函数

	 inline void push_strs(std::vector<std::string>& strs)
    {
    }

    template <typename T, typename... Args>
    inline void push_strs(std::vector<std::string>& strs, T&& v, Args&&... args)
    {
        auto str = (boost::format("%1%") % std::forward<T>(v)).str();
        strs.push_back(str);
        push_strs(strs, std::forward<Args>(args)...);
    }
    
这样
	
	template <typename... Args>
	C_Coder& operator()(Args&&... args)
	{
	    std::vector<std::string> vec;
	    push_strs(vec, std::forward<Args>(args)...);
	  	 xxx
	}
	
就可以将所有不定模板参数转成字符串数组。之后再使用 boost::join(vec, ", ") 就拼接好所有的函数参数。

这种不定模板的处理方式也很常见。先有一个默认函数，处理特殊情况，这相等于递归的终止条件。再有一个泛化的不定函数。比如

    void doIt(int a)
    {
    }
    
    template <typename T, typename... Args>
    inline void doIt(T&& v, Args&&... args)
    {
    	 doIt(std::forward<T>(v));
    	 doIt(std::forward<Args>(args)...);
    }
    
之后调用 

	doIt(1, 2, 3); // 1
	
这个函数有3个参数，就转化成

	doIt(1);
	doIt(2, 3);  // 2
	
再次展开成
	
	doIt(2);
	doIt(3);

有一个参数的函数为特殊情况，展开终止。相当于生成以下函数

	void doIt(int a)
	{
	}
	
	void doIt(int a, int b)
	{
		doIt(a);
		doIt(b);
	}
	
	void doIt(int a, int b, int c)
	{
		doIt(a);
		doIt(b, c);
	}
	
C++ 11 的这个特性，使得大量模板代码可以简化。std::forward 的作用是将模板参数完美转发，在模板代码中 std::forward 也很常见。具体可以再查查其它资料。

另外提一下，Swift 中也有类似的泛化不定参数，但它更加简洁，比如

	public protocol LayoutElement
	{
	 	xxx
	}
	
	public extension UIView
	{
		func xAlign(children: LayoutElement...) -> CGFloat
		{
		    return xAlign(children)
		}
	}
	
Swift 的泛化针对特定类型或协议。

## lamda函数和 DSL 收尾
lamda 函数在另外的语言中可能有不同的叫法，如匿名函数，代码块，闭包。lamda函数可以算是C++ 11中最大的改进。之前一系列 STL 中的算法，也变得容易使用。C++ 11 之前，大部分 STL 算法就是鸡肋。

我们写代码，有时会出现这种结构。

	// 做一些相同的准备事情
	doSomething
	// 做一些相同的收尾工作
	
这里的 domSomething 就可以传递一个lamda函数。

在定义 DSL 中，lamda函数也很有用，可以用来收尾。比如需要输出函数定义

	type funName(TypeA a, TypeB b, .....);
	
重载()运算符后，就可以使用合法的 C++ 代码表示成

	deffun("type", "funName")("TypeA", "a")("TypeB", "b")
	
但参数个数是不固定的，你事先不可能知道链式调用什么时候结束。这样就需要显式再调用一个收尾函数。比如
	
	deffun("type", "funName")("TypeA", "a")("TypeB", "b").finish()
	
定义DSL时，何时收尾的问题会不断出现（不单单是C++, 其它语言定义DSL时也会遇到收尾问题）。这种收尾在某种情况下可以使用lamda中解决。在lamda函数中填充，在主函数中收尾。比如函数定义可以写成：

    C_Coder& C_Coder::deffun(	const std::string& type, 
    								const std::string& name, 
    								const std::function<void(Args&)>& marker)
    {
        _buf += type;
        _buf += " ";
        _buf += name;
        Args args;
        marker(args);
        // 之后其实就是收尾工作
        br("(%1%)", boost::join(args._args, ", "));
        return *this;
    }

这样就用户就可以写出代码

	 coder.deffun("int", "max", [](auto& args) {
        args("int", "a")("int", "b");
    })
    
数组定义也是同样问题，比如需要输出

	int a[10] = {
		1, 2, 3, 4, 5,
		6, 7, 8, 9, 10
	};
	
直到最后才能输出最后的 }; 和计算出数组的个数。这种定义在 c_coder 中写成：

    defarray("int", "a", [](auto& make) { make
        (1, 2, 3, 4, 5)
        (6, 7, 8, 9, 10)
    })
   
make(1, 2, 3)(4, 5, 6) 这种语法只需要重载()运算符就行了。

另外因为收尾问题，a[1][2][4]; 这种语法也决定写成

	defarray("int", "a")(1, 2, 4);
	
而不直接写成
	defarray("int", "a")[1][2][4];
	
其实更好的写法是

	defarray("int", "a")[1, 2, 4];
	
但 C++ 的[]运算符重载只能传入一个参数。Swift 定义 subscript，也可以使用[]符号，并且可以传递多个参数。

我所知的，Lisp, Swift, C++, Ruby 这 4 种语言的语法特性都很适合写这种内部DSL。
    
    
	


	



	
	

	

	


	
	

