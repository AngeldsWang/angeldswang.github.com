---
layout: post
title: "Rejection Sampling"
date: 2014-08-11 03:30
comments: true
categories: [Sampling]
---

如何产生符合一定分布的随机变量？对于简单的随机变量，我们往往可以直接获得其概率分布函数$$F(x)=P(X\leqslant x)$$。但是对于更多复杂的随机变量来说，获得其概率分布的解析表达式是不可能的。对于这种情况，要产生这样的随机变量就需要另辟蹊径。Rejection sampling就是其中最具代表性的一种。下文主要参考了[Karl Sigman的notes](http://www.columbia.edu/~ks20/4703-Sigman/Monte-Carlo-Sigman.html "Karl Sigman's Lecture Notes on Monte Carlo Simulation")。

