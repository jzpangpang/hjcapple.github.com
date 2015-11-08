---
title: Bug 备忘：cocos2dx 中使用 lua 绑定调用 C++ 对象函数
layout: post
published: true

---

cocos2dx在使用的lua binding的时候，需要注意对象的生命期。今天遇到一个bug, 就跟生命期有关系。

程序中使用NodeUpdater来控制动画播放。在NodeUpdater的update函数中，根据传进来的时间，和动画的帧率，判断是否进入下一帧。原先有问题的lua代码是

	function M:update(time)
	    if self._node then
	        self._stepTime = self._stepTime + time
	        while self._stepTime >= self._frameTime do
	            self._node:stepFrame()
	            self._stepTime = self._stepTime - self._frameTime
	        end
	    end
	end

当update的时间过大时候，那个while循环有时可能连续执行两次或者多次stepFrame。

假设连续执行两次，

	第一次stepFrame, 进入第10帧。
	第二次stepFrame, 进入第11帧。

当代码调用动画节点的stepFrame的时候，会触发 listener 的 onAnimationStep 回调。这样在第一次stepFrame调用中，就有可能在回调中间接删除了动画节点，之后在while循环中执行第二次stepFrame，因为动画节点已经被删除，就引起崩溃。

在iPhone 5S, iPhone 6，或者在Mac机上测试时，机器的性能很快。update中传进来的时间间隔往往比较小，while循环中只会调用一次stepFrame，上面代码的隐含错误并没有展示出来。而在性能比较慢的机子，比如在iPad 3上面测试时，有时帧率下降，update中传进来的时间间隔就比较大，while循环中就有可能调用多次stepFrame。这样就出现了问题。

知道了原因，修改就很容易。

	function M:update(time)
	    if self._node then
	        self._stepTime = self._stepTime + time
	        -- retain, 再release, 防止中途节点被删除
	        self._node:retain()
	        while self._stepTime >= self._frameTime do
	            self._node:stepFrame()
	            self._stepTime = self._stepTime - self._frameTime
	        end
	        self._node:release()
	    end
	end

上面在执行while循环之前，使用retain，release来保证中途节点不会被删除。更进一步设置一个标记值，当有这个标记值后，就不再执行stepFrame。

	function M:update(time)
	    if self._freeze or self._node == nil then 
	        return
	    end
	
	    -- retain, 再release, 防止中途节点被删除
	    self._node:retain()
	    while self._stepTime >= self._frameTime do
	        if self._freeze then 
	            break
	        end
	        self._node:stepFrame()
	        self._stepTime = self._stepTime - self._frameTime
	    end
	    self._node:release()
	end

上面代码，在原来C++的版本中并没有问题。原来的C++版本，使用了智能指针，当赋值的时候，自动调用了retain，在析构时候再进行release。因此外部小心引用计数的话就不会删除了对象。

