---
title: Swift tip：将 dispatch_async 的回调转成串行
layout: post
published: true
---

在 iOS/Mac 编程中，经常使用 Grand Central Dispatch，它的语法使用回调。如：

```Swift
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), {
    print("This is run on the background queue")

  let result = "hello, World"
    dispatch_async(dispatch_get_main_queue(), {
        print("This is run on the main queue, after the previous block")
    print(result)
    })
})
```

这样就会产生一系列嵌套，当嵌套太多的时候代码就会混乱。

我编写了一个简单的库 [Async](https://github.com/hjcapple/Async)，将回调转换成串行。比如上面代码就可以转换成：

```Swift
Async.background {
    print("This is run on the background queue")
    return "hello, World"
}.main { (result : String) in
    print("This is run on the main queue, after the previous block")
    print(result)
}
```

具体的实现可以看代码，所有的代码不到 100 行。思路可以稍微参考这篇文章，[轻松无痛实现异步操作串行](https://zhuanlan.zhihu.com/p/20780839)。

在 GitHub 上其实是有一个 [https://github.com/duemunk/Async](https://github.com/duemunk/Async) 的，它的问题是不能捕获上一步骤的返回结果。

