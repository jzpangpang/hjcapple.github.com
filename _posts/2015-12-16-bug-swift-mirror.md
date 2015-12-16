---
title: Bug 备忘：Swift 中使用 Mirror 引起的内存泄露
layout: post
published: true
---

现在网络 API 普通使用 Json 接口，向服务器发送 Json 数据，也返回 Json 数据。这样就需要将程序中的对象转成 Json 或者将 Json 转成对象。

我写了一个序列化模块，利用 Swift 中的反射，将对象和字典相互转化。而字典和 Json 数据相互转化已经有系统函数。其实 Swift 中的反射现在还挺弱的，不能处理嵌套的结构。有时需要覆写一个函数，一层层处理。但现在暂时也够用了。

昨天遇到一个 Bug。程序定时发送心跳包，会有内存泄露，每发送一次就泄露一次。经过调试，初步确定是序列化模块有问题。

我新建另外的工程，将跟序列化无关的代码去掉，另外开了一个定时器不断地将对象转成字典。这样就可以不断重复泄露现象。一直简化，发现下面的代码有泄露：

	func objectFromAny(any: Any) -> AnyObject?
	{
	    let mirror = Mirror(reflecting: any)
	    if let style = mirror.displayStyle
	    {
	        switch style
	        {
	        case .Optional:
	            guard let (_, some) = mirror.children.first else { return nil }
	            return objectFromAny(some)
	            
	        case .Enum:
	            guard let value = any as? ModelRawInt else { return nil  }
	            return NSNumber(integer: value.rawValue)
	            
	        default:
	            break
	        }
	    }
	    if let str = any as? String
	    {
	        return str
	    }
	    return numberFromAny(any)
	}
	
但奇怪的是，我观察代码，一直找不到有什么问题。后面再将代码简化：

    let details = NSMutableDictionary()
    details["ConversationId"] = "Hello"
    
    let dict : AnyObject? = details
    let mirror = Mirror(reflecting: dict)
    if let style = mirror.displayStyle
    {
        switch style
        {
        case .Optional:
            guard let (_, some) = mirror.children.first else { return }
            if let some = some as? NSNumber
            {
            }
            
        default:
            break
        }
    }
    
上面代码会有内存泄露。但是一旦将那句， 

	if let some = some as? NSNumber 
		
中那个 NSNumber 修改成 String，改成

	if let some = some as? String
	
就没有问题。我应该是踩到 Swift 的坑了。推测是 details 是 Objective-C 的字典类，当一旦 Mirror 就会转成 Swift 的字典类。但 some 中使用 some as? NSNumber 就又会转成 Objective-C 的字典类。这样转换几次就出问题了。似乎 Swift 跟 Objective-C 中字典和数组的转换，并非简单的转换容器指针。而是会将容器里面的元素也一个个转换的。

现在 Swift 跟 Objective-C 的相互交换还有不少问题。最后修改方法，我先判断一些是否是字典，是字典的话就直接退出。之后程序在单独处理字典。反正现在序列化模块，还不能处理对象中嵌套对象、或者嵌套字典、数组之类的情况。

	func objectFromAny(any: Any) -> AnyObject?
	{
	    let mirror = Mirror(reflecting: any)
	    if let style = mirror.displayStyle
	    {
	        switch style
	        {
	        case .Dictionary, .Collection:
	            return nil
	            
	        case .Optional:
	            guard let (_, some) = mirror.children.first else { return nil }
	            return objectFromAny(some)
	            
	        case .Enum:
	            guard let value = any as? ModelRawInt else { return nil  }
	            return NSNumber(integer: value.rawValue)
	            
	        default:
	            break
	        }
	    }
	    if let str = any as? String
	    {
	        return str
	    }
	    return numberFromAny(any)
	}
    





