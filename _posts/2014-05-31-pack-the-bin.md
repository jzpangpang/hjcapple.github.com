---
title: 一千种放箱方法 -- 解决二维矩形装箱问题的的实用方法
layout: post
published: false

---

作者: Jukka Jylänki

时间: 2010年二月27日


摘要
-----
我们回顾了几种算法，能够被用于解决，将矩形放到二维有限箱子的问题。大多被提到的想法，已经在学术上被仔细研究过，但有些变种方法比较少人知道。有些显然被认为是“民间口述”，之前的参考也没有提到。不同的变种被提到，并且相互比较。这项研究的主要贡献是，从解决二维有限装箱问题的观点，原创了，将不同的变体方法分类。这项工作主要讲述，经验的研究，在这个问题的变体，矩形是否需要正交摆放，是否可以旋转90度。合成的测试，被用于主要的参照，解决一个现实中的特定问题，生成纹理集，测试每个方法，在现实世界中的性能。 As a related contribu- tion, an original proof concerning the number of maximal orthogonal rectangles inside a rectilinear polygon is presented.

关键词: 二维装箱，优化，启发式算法，即时算法，NP难度


目录
--------













































1 介绍
--------
二维矩形装箱问题是，组合优化上的一个经典问题。这个问题，是给出一系列的矩形。(R1,R2,....,Rn)，Ri = (wi, hi)。任务是找出如何摆放这些矩形，使得箱子的尺寸最小。(W, H)。并且两个矩形没有相交，也不能一个矩形包含另一个矩形。这个问题，有不少现实世界的应用。并被证明是一个NP难度。可以被归约成，二维划分问题。并不存在多项式渐进时间逼近策略(APTAS), 它是一个APX-难度。已经做了很多研究工作，发展出高效的启发式算法，去逼近最优解法。在这项研究，我们提供了几种不同的算法，并比较了他们在实际中的性能。修改算法的一个启发式的决定过程，在真实的情况下，可以得到十分不同的结果。












