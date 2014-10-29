---
layout: post
title: "Lowest Common Ancestor Problem"
date: 2014-10-29 20:51
comments: true
categories: [Algorithm]
---

最近公共祖先问题是一个很经典的树上的问题。目的就是找到距离2个节点最近的祖先节点。先抛开算法本身，这个问题有什么意义呢？[Alfred Aho，John Hopcroft，和Jeffrey Ullman](http://dl.acm.org/citation.cfm?id=804056)这几位大佬在70年代提出这些问题的时候勉为其难地想到了这样一个无聊的应用--["宗谱服务"](http://hihocoder.com/problemset/problem/1062)。大意就是有一个在线网站不断的获得一些家族成员之间的祖辈信息，并对任意两个成员提供查询他们最近公共祖先的服务。其实在70年代的时候，编译器设计优化是十分前沿的研究方向，很多算法都是为此服务的。这个最近公共祖先的问题实际上是作为编译器设计中一个很经典的问题--[在有向图上查找Dominator节点][ref1]的一个子问题被引起广泛关注。不能再扯更多了，其他的应用比如版本控制什么的可以详见[stackoverflow上的讨论][ref2]。
[ref1]: http://en.wikipedia.org/wiki/Dominator_(graph_theory)
[ref2]: http://stackoverflow.com/questions/3542452/what-are-the-practical-applications-of-the-lowest-common-ancestor-algorithms

