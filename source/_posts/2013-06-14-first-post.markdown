---
layout: post
title: "Indian Buffet Process"
date: 2013-06-14 17:46
comments: true
categories: [Machine Learning, nonparameteric Bayes]
---

The Indian buffet process is a **stochastic process** defining a **probability distribution** over **equivalence classes** of sparse binary matrices with a finite number of rows and an unbounded number of columns. ([Griffiths et al., 2011](http://www.jmlr.org/papers/volume12/griffiths11a/griffiths11a.pdf "Griffiths and Ghahramani"))  上面这段话就是Indian buffet process (IBP)的fathers给出的最简单明确的说明。首先IBP是一个随机过程，那么到底什么是随机过程，随机过程又和概率分布有什么关系，等价类又是什么？这里先从随机过程说起。

#随机过程
在定义随机过程之前首先要明确两个概念。第一个是**概率空间(probability space)**，在定义概率空间之前还要明确另一个概念**可测空间(measurable space)**。当然这一切还要从一个叫“$\sigma$-代数”的定义说起
