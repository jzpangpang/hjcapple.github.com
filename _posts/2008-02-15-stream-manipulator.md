---
title: 写自己的C++流操纵符
layout: post
published: true

---

学C++，肯定用过操纵符(manipulator)，或者之前没有听过这个名词，也在不知不觉中用过。C++ 标准库中的操纵符有很多，常用的有

* std::endl
* std::hex
* std::setw
* std::setfill。

有些操纵符没有参数，有些有一或多个参数。没有参数的操纵符包含iostream头文件就可以使用。有参数的操纵符用含头文件iomanip, 这里 io 指 input/output(输入输出), manip是manipulator的头几个字母。

利用操纵符有时可以使程序简单直观。除了利用内部定义好的，我们也可以写自己的操纵符。

无参数操纵符
--------
如何写呢? 首先看看没有参数的。这种操纵符通常可以作为函数来写。比如，我想写操纵符来输出日期和时间，可以这样做。

	std::ostream& time(std::ostream& ostr)
	{
	    time_t curtime;
	    time(&curtime);
	    tm* ptime = localtime(&curtime);
	
	    ostr << ptime->tm_hour << ':' << ptime->tm_min << ':' << ptime->tm_sec;
	    return ostr;
	}
	
	std::ostream& date(std::ostream& ostr)
	{
	    time_t curtime;
	    time(&curtime);
	    tm* ptime = localtime(&curtime);
	
	    ostr << (ptime->tm_year + 1900) << '-' << (ptime->tm_mon + 1) << '-'
	         << ptime->tm_mday****;
	    return ostr;
	}

之后就可以用了，用法和 std::endl 一样。

	int main()
	{
	    std::cout << date << std::endl;
	    std::cout << time << std::endl;
	    return 0;
	}

time 和 date 可以作为输出使用，如果你写的操纵符要作为输入，函数原形相应为

	std::istream& fun_name(std::istream& istr)

如果可以用作输出输入，可以写为

	std::ios& fun_name(std::ios& iostr);

为什么写成这样就可以用的呢? 我以输出为例说一下。
C++库里面有个函数为

	std::ostream& operator << (std::ostream& ostr, std::ostream& (*pf)(std::ostream&), 

pf为函数指针，注意指针类型和我们写的 time 和 date 一样。它的实现可以简单的为 return pf(ostr);。

这样，我们写 `std::cout << date` 就调用了

	 operator << (std::cout, date)
 
跟着转为 `date(cout)`, 因为返回值为 `std::cout`, 可以和 `std::endl` 连用。

有参数操纵符
--------
再来看看有参数的操纵符，这个不能作为函数来写。举个例子, 我想有个操纵符来将某个字符c重复n次，repeat(char c, int n), 当n为负或0时无效。可以这样做。

	class repeat
	{
	public:
	    repeat(char c, int n) : _c(c), _n(n)
	    {
	    }
	    
	    friend std::ostream& operator<<(std::ostream& ostr, const repeat& r)
	    {
	        for (int i = 0; i < r._n; i++)
	        {
	            ostr << r._c;
	        }
	        return ostr;
	    }
	    
	private:
	    char _c;
	    int _n;
	};****


之后可以这样使用

	int main()
	{
	    std::cout << repeat('#', 13) << std::endl;
	    return 0;
	}

这个其实也很简单，`repeat('#', 13)` 创建了一个临时的对象tmp, 类型为repeat, 跟着调用 std::cout<<tmp, 也就是调用

	operator << (std::cout, tmp)

也就是创建了一个可流的对象。有个地方要注意，

	friend std::ostream& operator << (std::ostream& ostr, const repeat& r)

中的const不能去掉，不然就不能创建临时对象了。


注
--------
1. C++ 标准库其实没有一个叫 std::ostream 的类，只有个 std::basic_ostream，std::ostream只是其 typedef 定义出来的。std::basic_ostream是个模板类。

2. localtime 函数返回的 tm 结构，字段 tm_year 并不是真实年份，而是真实年份减去1900年，tm_mon取值为0-11, 所以要算出年和月用相应加上1900和1。

3. operator << 为左结合，当写 

		std::cout << date << std::endl; 
其实为

		((std::cout << date) << std::endl); 
`std::cout << date` 返回`std::cout`, 跟着再执行`std::cout << std::endl;`<br>
这就解释了为什么我们写函数 operator << 的时候为什么返回值一为`std::ostream&`, 这是为了可以连续输出。









