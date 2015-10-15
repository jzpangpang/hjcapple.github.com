---
title: Bug 备忘：OpenGL ES 2.0下纹理显示成黑色
layout: post
published: true
---

在用iOS的OpenGL ES 2.0渲染纹理，纹理总是显示成黑色。而同样的代码在Mac OS X上是没有问题的。

用宏一行行测试：

	#define CHECK_GL_ERROR()                                                               \
	    (                                                                                  \
	        {                                                                              \
	        GLenum __error = glGetError();                                                 \
	        if (__error)                                                                   \
	            printf("OpenGL error 0x%04X in %s %d\n", __error, __FUNCTION__, __LINE__); \
	        })

其中代码：

	glEnable(GL_TEXTURE_2D);
	CHECK_GL_ERROR();
	
报告错误

	OpenGL error 0x0500 in -[PainterView onInit] 145
	
0x0500错误为`GL_INVALID_ENUM`，枚举值不合法。在GL 2.0下，有其它的方式来指示使用纹理，不用调用`glEnable(GL_TEXTURE_2D)`。

再次一行行检查，所有的gl代码都没有报错，但是纹理仍然显示成黑色。

上网找了个很基础的显示图片的代码，它的代码可以显示图片。我一行行对照，没有发现有什么不同。折腾了挺久的，最终在：

[OpenGL ES 2.0 Texture mapped, but shows as black](http://stackoverflow.com/questions/11557382/opengl-es-2-0-texture-mapped-but-shows-as-black)

找到一段话：

> In OpenGL ES 2.0, textures can have non-power-of-two (npot) dimensions. In other words, the width and height do not need to be a power of two. However, OpenGL ES 2.0 does have a restriction on the wrap modes that can be used if the texture dimensions are not power of two. That is, for npot textures, the wrap mode can only be GL_CLAMP_TO_EDGE and the minifica- tion filter can only be GL_NEAREST or GL_LINEAR (in other words, not mip- mapped). The extension GL_OES_texture_npot relaxes these restrictions and allows wrap modes of GL_REPEAT and GL_MIRRORED_REPEAT and also allows npot textures to be mipmapped with the full set of minification filters.
I hope this helps.

我先将图片尺寸修改成 512 * 512（原图为500 * 500), 图片可以显示出来了。之后将图片还原成500 * 500, 在生成纹理的时候设置：

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	
终于正常了。我现在才发现，上网找的例子，图片是2的N次方，所以就算没有设置`GL_CLAMP_TO_EDGE`也可以正常显示，怪不得我找不出跟例子代码有什么不同。

我之前知道iOS的OpenGL ES 2.0是支持 non-power-of-two 纹理的，所以一开始没有考虑到图片大小，但我不知道在gles下，会有一定约束。

总结:
-------
* OpenGL ES 2.0，假如图片不是2的N次方，需要将WRAP方式设置成`GL_CLAMP_TO_EDGE`（或者其它方式，但不能使用默认的）。

* OpenGL ES 和 OpenGL 会有微妙的不同。同一代码，有可能在Mac OS X上没有问题，iOS上有问题。

* 遇到问题，可以先上网查查。

* Google很好用，百度和Bing在找资料方面都不够好。但Google被封了。

* 对照代码时候，最好让资源也相同。

	