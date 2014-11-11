---
title: C++的多态威力，结合分离的类
layout: post
published: true

---

###注
这篇文章比较早期的了，有些做法现在看起来不算很好。
现在我倾向直接使用组合，将不可修改的类当成一个成员变量，当成实现的一部分。而非用适配器来继承。下面的做法还不够直接简单，指针管理方面有些问题。

###*
编程的时候，应该将类和函数的实现细节隐藏起来，给用户一个统一的接口。基本上，很多设计模式，很多技巧都是为了达到这个目的。如果可以隐藏细节，给出一个统一的接口，你这个设计多数是好的。

考虑两个类 MouseMessage, KeyMessage, 他们的定义如下：

###MouseMessage
		class MouseMessage
		{
		public:
		    MouseMessage()                { message = "";  }
		    MouseMessage(string str)        { message = str; }
		    void setMessage(const string& str)    { message = str; }
		    void writeMessage(ostream& out) const;
		private:
		    string message;
		};
		
		void MouseMessage::writeMessage(ostream& out) const
		{
		    out<<message<<endl;
		}
		
###KeyMessage

		class KeyMessage
		{
		public:
		    KeyMessage()            { message = "";  }
		    KeyMessage(string str)        { message = str; }
		    void set(const string& str)    { message = str; }
		    friend ostream& operator<<(ostream& out, const KeyMessage& keyMsg);
		private:
		    string message;
		};
		
		ostream& operator<<(ostream& out, const KeyMessage& keyMsg)
		{
		    out<<keyMsg.message<<endl;
		    return out;
		}
		
这两个类都完成了设置和打印消息的功能，但他们给出的接口，或者说函数的调用方法不一样。如果这两个类是你自己写的，应该修改接口，使他们函数调用方法一致。

问题就在于，他们是由不同的人写的，你得到的只是编译好的二进制文件，你不可以修改他们的源代码。不过他们有你想要的功能，所以你想利用他们的功能，节省自己重新编写代码的时间。

现在你就有个目的，利用已有的类，并且使自己利用它们的方式一致。你可以这样做，写几个简单的函数：

		inline void set(KeyMessage& keyMsg, const string& str)
		{
		    keyMsg.set(str);
		}
		
		inline void set(MouseMessage& mouseMsg, const string& str)
		{
		    mouseMsg.setMessage(str);
		}
		
		inline void write(ostream& out, const KeyMessage& keyMsg)
		{
		    out<<keyMsg;
		}
		
		inline void write(ostream& out, const MouseMessage& mouseMsg)
		{
		    mouseMsg.writeMessage(out);
		}

这个时候，因为函数是重载的，你可以统一的调用方法set，和write。

		MouseMessage msg1("Mouse Move");
		KeyMessage   msg2("Key Pressed");
		
		set(msg1, "Mouse Drag");
		set(msg2, "Key Up");
		
		write(cout, msg1);
		write(cout, msg2);

这样调用起来，没有什么面向对象的感觉。MouseMessage和KeyMessage都是消息，希望可以将他们都放在一个向量或者数组里面，统一遍历。很可惜，这两个类是不一样的，你不可以把他们放在一起。它们也不是在同一个类中派生出来的，所以你也不可以将它们的指针放在一起。


可以使用下面的方法，写一个抽象类，名字就叫做 Message

		class Message
		{
		public:
		    void virtual write(ostream& out) = 0;
		    void virtual set(const string& str) = 0;
		};
		
跟着，使用一个适配器

		template <typename T>
		class MessageAdapter : public Message
		{
		public:
		    MessageAdapter(T* pt)    { ptMsg = pt; }
		    void virtual write(ostream& out)    { ::write(out, *ptMsg); }
		    void virtual set(const string& str)    { ::set(*ptMsg, str);    }
		private:
		    T* ptMsg;
		};


再写一个创建函数

		template <typename T>
		Message* newMsg(T& msg)
		{
		    return new MessageAdapter<T>(&msg);
		}


现在你可以写出这样的代码了

		MouseMessage msg1("Mouse Move");
		KeyMessage   msg2("Key Pressed");
		
		vecotr<Message*> v;
		v.push_back(newMsg(msg1));
		v.push_back(newMsg(msg2));
		
		for (int i=0; i<v.size(); i++)
		{
		    v[i]->write(cout);
		}

这是模板、虚函数和函数重载的结合。模板根据你的调用类型，推断出你用的是 MouseMessage 还是KeyMessage, 跟着 new 出一个MessageAdapter。

MessageAdapter 派生自 Message，所以可以用 Message 指针统一调用。根据虚函数的作用，Message指针调用 MessageAdapter 的 write 和 set 方法。因为 MessageAdapter 也是模板类，它也可以推断出你用的是 MouseMessage 还是 KeyMessage，跟着调用了重载的全局函数 write 和 set。

不过这时候，有点不足，你不可以写出这样的代码：

v.push_back(newMsg(MouseMessage("Mouse Drag")));
v[0]->write(cout);

因为 newMsg 的返回是个指针，MouseMessage("Mouse Drag")作为参数只不过是个临时变量，之后就注销了，既然没有对象，v[0]->write(cout); 就没有意义了。所以你想将 Message放进向量，一定要先定义一个对象。这样太不合理了，你没有理由想用向量存储100个指针，就要定义100个对象，这个是不可以忍受的。所以可以在定义一个函数：

		template <typename T>
		Message* newMsg(T* ptMsg)
		{
		    return new MessageAdapter<T>(ptMsg);
		}


注意，这个函数的参数是个指针。要是你传递的是指针就会调用这个函数，要是你传递的是数值，就会调用先前的那个函数。
从此之后，你可以写出这样的代码：

		MouseMessage msg1("Mouse Move");
		KeyMessage   msg2("Key Pressed");
		
		vector<Message*> v;
		v.push_back(newMsg(msg1));
		v.push_back(newMsg(msg2));
		
		v.push_back(newMsg(new MouseMessage("Mouse Dray")));
		v.push_back(newMsg(new KeyMessage("Key Up")));
		
		for (int i=0; i<v.size(); i++)
		{
		    v[i]->write(cout);
		}


无论你传递的是指针，还是传值，都可以了，MouseMessage 和 KeyMessage 也可以用相同的方式调用。

在《面向对象编程导论》 (美)Timothy A.Budd 著 有相似的例子。它给出的例子是出现两个类 Orange 和Apple，抽象类为 Fruit。我改成 Message，也修改了另外一些地方。不过原理是一样的，建议看看。那本书是将很多面向对象语言进行比较的, 有 Java, C++, C#, Smalltalk 等等。

后来补充
------------
上面的代码有些问题，

* vector 里面存储的是指针，如果 msg1 或 msg2 过早的析构了，那么指针就垂悬了；
* 还有一个则是 v.push_back(newMsg(new MouseMessage("Mouse Dray"))); 这样产生的 Message 不能 delete。

可以将上面的代码，裸指针修改成智能指针。std::shared_ptr来解决释放问题。

