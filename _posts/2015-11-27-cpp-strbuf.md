---
title: 转换电子书中的 html
layout: post
published: true
---

## 显示电子书
我之前研究 mobi 电子书的格式。mobi 电子书内部只是封装了一些 html，其它电子书也是一样。因此电子书的显示，很大程度上可以归结成网页的处理。

显示一个电子书，仅仅考虑最简单的显示而不考虑分页等等。最简单的方式是抽取其中的 html，转成标准的网页格式，再用系统的浏览器来读取。电子书封装的 html 会有一些特殊的标签，图片、链接等表示的方式跟标准的网页会有所不同，因此需要转换一下。

比如图片就可以转成 base64 的格式，之后用网页的 src = "data:image/jpg;base64:xxxx" 的方式将图片数据直接嵌入到网页中，这样可以简化很多的代码。另外更常规的方式，是为电子书注册自己的代理协议，让浏览器发起资源请求，接收到特定的协议后就从电子书中抽取对应的资源。

转换过程当中也可以嵌入自定义的 css 文件，这样就可以按照预览定义好的格式来美化输出。

将上面的描述简化，主要有 3 个过程：

1. 解析原始的电子书 html，变成一棵 DOM 树。
2. 修改转换 DOM 树。
3. 将 DOM 树转成标准的网页 html, 送到浏览器中。

这 3 个过程，都涉及到字符串处理。其实也可以将第 2 和第 3 个过程混在一起，一边处理结点一边输出。但这样的话，输出过程就不可以在另外的程序中复用了。分开之后，1、3 这两个过程是通用的。

解析 html, 可以使用 gumbo 库。gumbo 是一个纯 c 的 html 解析库，但它并没有修改 DOM 树的接口。这时可以简单转换一下，将 gumbo 库中的节点转换成自定义的节点，就可以得到一个可以操作的 DOM 树。

网页的节点（HtmlNode)，可以大致分成两大类，一是 Element, 可以有属性也有子节点，一是 Text，只包含文字。Comment 等可以归结成文字。

习惯面向对象思维的，这时会自然地将定义一个基类`HtmlNode`, 之后再定义子类 `HtmlTextNode`, `HtmlElementNode`，`HtmlCommentNode` 等等。但这里其实只需要定义一个 `HtmlNode` 类就够了, 之后用 `type` 字段来区分不同的子节类型。用同一个节点类有个好处，我们可以定义一个内存池（`boost`已经有了），将所有节点都从内存池中分配和释放。这样节点的分配和释放就会很快。

转换之后就可以根据具体应用，自由操作 DOM 树了。

第 3 步是将 DOM 树，转换成标准的网页 html。

html 是字符串。C++ 中处理字符串很自然会使用 `std::string`。在多语言处理中，假如内部编码不使用 utf8 ，简直是自己找麻烦，utf8 字符串就可以使用 std::string 来存放。最开始我也是采用 `std::string`, 后面经过测试修改成使用 `std::vector<char>`。

将 DOM 树转成 html，并不用涉及到字符串的分割、合并、比较大小等操作。而仅仅需要一个字符缓冲区，这缓冲区一直都是从最后放字符。这种情况下，`std::vector<char>` 比 `std::string` 有优势了。

`std::string` 因为需要兼容 C 字符串，很多实现中，每次从最后放字符，总需要维护最后的空字符 0。另外 `std::string` 并没有类似 `std::vector<char>` 的 `reserve` 这个函数。在可以大致估计出最终字符长度的情况下，使用 `reserve` 预先保留一定空间，可以大大避免内存重分配的次数。比如这里处理电子书 html 的情形下，一开始就分配原始 html 的 1.5 倍内存空间，在转换过程中，就基本不会重新分配内存。

这里将 `std::string` 修改成 `std::vector<char>`，代码写起来、维护起来成本是差不多的，但速度就快了很多。懂得多一点，稍微注意一下，写出运行效率高的程序的成本其实跟效率低的成本差不多，但会对程序员的要求会更高一些。

很多人会觉得现在的机器下，性能已经不重要了，随便写都可以。但我的观点是，假如写起来不麻烦并且以后维护起来也差不多，当然是怎么快怎么写，而假如需要付出很大努力才可以快上一点点，就不值得了。

软件设计的不同，可以使性能有数量级的差别，而软件的设计也很大程度上归结成数据结构的设计。而在相同的软件设计下，对于相同含义的代码的不同写法，也往往可以导致几倍的效率差别。假如一开始随便写不注意的话，程序慢了，到最后再怎么优化，也只能从蚂蚁爬变成乌龟爬。而一开始就注意，可以从跑车的速度提升到飞机的速度。

## 性能优化例子
最高效率的优化是软件设计，而我们现在常说的优化只是细节的调整。

细节优化第一准则是需要测量不能乱猜。有很多工具可以测量程序的时间，只有有这个观念，自然会找到方法。

举个例子，比如我现在打开电子感觉慢了，需要优化打开速度。上面说的将 std::string 修改成 std::vector<char> 是优化手段之一，但还是想更快。

单次测量有时会有误差，我们需要累计时间。这里就应该反复打开电子书，可以将打开书的过程写成一个函数，之后可以写一个循环或设置定时器，不断地打开、关闭电子书。这样程序的所有时间就会耗在这个过程，一些误差就会被过滤掉。

一段时间之后，再看结果。耗时最多的就是程序热点，具体可以定位到某个函数。修改这个函数之后再测量，修改之后之前耗时最高的函数往往就不是最高。这时耗时此高的函数会成为下一个热点。这样循环三四次，就可以有明显的效率优化。

经过测试，我发觉 `write_html_text` 这函数有点耗时：

	static inline void strbuf_push(std::vector<char>& result, const std::string& str)
	{
	    result.insert(result.end(), str.begin(), str.end());
	}
	
	static void write_html_text(std::vector<char>& result, const std::string& text)
	{
	    for (auto c : text)
	    {
	        switch (c)
	        {
	            case '<':
	                strbuf_push(result, "&lt;");
	                break;
	
	            case '>':
	                strbuf_push(result, "&lt;");
	                break;
	
	            case '&':
	                strbuf_push(result, "&lt;");
	                break;
	
	            default:
	                result.push_back(c);
	                break;
	        }
	    }
	}
	
html 有些特殊的字符需要转换，所以输出文字的时候需要检查是否有特殊字符。接下来就具体分析这个函数有没有什么地方可以改进。首先发现一个问题，const char* 会被隐式转成 std::string, 就多了一次复制。就顺手添加上：

	 static inline void strbuf_push(std::vector<char>& result, const char* str)
    {
        result.insert(result.end(), str, str + strlen(str));
    }
    
这样的话，`strbuf_push(result, "&lt;");`其中的`"&lt;"`就不会转成std::string。但似乎在运行时候再算strlen(str)也有点浪费，这个常量字符串的长度一开始就应该知道了。这样就修改成:

	static inline void strbuf_push(std::vector<char>& result, const char* str, size_t size)
	{
	    result.insert(result.end(), str, str + size);
	}
	
	#define STRBUF_PUSH_LITERAL(result, cstr) strbuf_push(result, cstr, (sizeof(cstr) / sizeof(char)) - 1);
	
	static void write_html_text(std::vector<char>& result, const std::string& text)
	{
	    for (auto c : text)
	    {
	        switch (c)
	        {
	            case '<':
	                STRBUF_PUSH_LITERAL(result, "&lt;");
	                break;
	
	            case '>':
	                STRBUF_PUSH_LITERAL(result, "&gt;");
	                break;
	
	            case '&':
	                STRBUF_PUSH_LITERAL(result, "&amp;");
	                break;
	
	            default:
	                result.push_back(c);
	                break;
	        }
	    }
	}
宏`STRBUF_PUSH_LITERAL`可以在编译时候就知道常量字符串的长度，就避免一次计算。修改之后再测试似乎也没有快多少。再次观察。这里每次处理一个字符，而特殊字符是比较少出现的，就导致每次只往 result 中塞一个字符。经分析后，再修改成：

	 static inline void strbuf_flush(std::vector<char>& result, const std::string& text, size_t i, size_t& last)
    {
        assert(i >= last);
        if (i > last)
        {
            strbuf_push(result, &text[last], i - last);
            last = i + 1;
        }
    }

    static void write_html_text(std::vector<char>& result, const std::string& text)
    {
        size_t last = 0;
        size_t i;
        for (i = 0; i < text.size(); i++)
        {
            char c = text[i];
            switch (c)
            {
                case '<':
                    strbuf_flush(result, text, i, last);
                    STRBUF_PUSH_LITERAL(result, "&lt;");
                    break;

                case '>':
                    strbuf_flush(result, text, i, last);
                    STRBUF_PUSH_LITERAL(result, "&gt;");
                    break;

                case '&':
                    strbuf_flush(result, text, i, last);
                    STRBUF_PUSH_LITERAL(result, "&amp;");
                    break;

                default:
                    break;
            }
        }
        strbuf_flush(result, text, i, last);
    }
    
这里的思路是将非特殊字符的字符累积起来，批量放进`reulst`中。经过测量，`write_html_text`已经够快了。也没有什么地方可以明显修改的，就可以停手了。

现在特殊字符只有 3 个，当特殊字符增多的时候，switch (c) 里面的语句就会变乱，这样就需要将 switch (c) 中的语句抽取出来进行重构，可以将特殊字符放在表格中进行查找。但现在直接用 switch 就足够好了，当以后出现新情况时候再进行修改。

	






