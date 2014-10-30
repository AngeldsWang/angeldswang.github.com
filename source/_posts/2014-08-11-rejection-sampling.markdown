---
layout: post
title: "Rejection Sampling"
date: 2014-08-11 03:30
comments: true
categories: [Sampling, Monte Carlo, Statistical Methods]
---

如何产生符合一定分布的随机变量？对于简单的随机变量，我们往往可以直接获得其概率分布函数$$F(x)=P(X\leqslant x)$$。但是对于更多复杂的随机变量来说，获得其概率分布的解析表达式是不可能的。对于这种情况，要产生这样的随机变量就需要另辟蹊径。**Rejection sampling**就是其中最具代表性的一种。下文主要参考了[Karl Sigman的notes](http://www.columbia.edu/~ks20/4703-Sigman/Monte-Carlo-Sigman.html "Karl Sigman's Lecture Notes on Monte Carlo Simulation")。<!--more--> 

Rejection sampling的主要思想是用另一个容易产生的概率分布$$G$$，(它的概率密度是$$g(x)$$)，来尽可能的逼近所求的概率分布$$F$$。形式化出来就是，这两个概率分布密度的比有明确的上界$$c$$，即$$\text{sup}_x\{f(x)/g(x)\}\leqslant c$$。实际中，$$c$$的值尽可能的接近$$1$$。下面首先来看如何用Rejection sampling来获得连续随机变量。对于离散随机变量，情况基本类似。

###<font color='#FF5544'>用Rejection sampling算法产生连续随机变量</font>
<table border='2' cellpadding='15px' bordercolor='#FF5544'>
<td>
<ol>
<li>生成服从概率分布$G$的随机变量$Y$。</li> <p/>
<li>从均匀分布$Uniform(0,1)$采样获得一个随机变量$U$(和$Y$相互独立)。</li> <p/>
<li>If $U\leqslant \frac{f(Y)}{cg(Y)}$，then $X=Y$("accept");<br/>
    Otherwise goto step 1 ("reject")。</li> 
</ol>
</td>
</table>
<br/>
在证明上述算法之前，有以下几点值得我们注意：  


*   $$f(Y)$$和$$g(Y)$$都是随机变量，因此$$\frac{f(Y)}{cg(Y)}$$也是一个随机变量。这一比值和step 2 中的随机变量$$U$$是相互独立的。  
*   这一比值是以0和1为上下界的，即$$0\leqslant \frac{f(Y)}{cg(Y)}\leqslant 1$$。  
*   step 1 和 step 2 调用的次数$$N$$(也就是成功采样获得一个$$X$$所需的迭代次数)本身也是一个服从几何分布的随机变量。其一次试验就发生的概率$$p=P(U\leqslant \frac{f(Y)}{cg(Y)})$$，则$$P(N=n)=(1-p)^{n-1}p, n\geqslant 1$$。因此迭代次数的期望为$$E(N)=1/p$$。  
*   最终，我们可以将所期望获得的随机变量$$X$$的概率分布$$F$$等价于在事件$$\{U\leqslant \frac{f(Y)}{cg(Y)}\}$$发生的条件下，随机变量$$Y$$的概率分布。  
<br/>
另外，若以随机变量$$Y$$为条件，事件$$\{U\leqslant \frac{f(Y)}{cg(Y)}\}$$发生的概率为：$$P(U\leqslant \frac{f(Y)}{cg(Y)}|Y=y)=\frac{f(y)}{cg(y)}$$。考虑到$$Y$$的概率密度为$$g(y)$$，通过去条件化并对$$Y$$所有可能的值上进行积分，这样就可以得到$$p=P(U\leqslant \frac{f(Y)}{cg(Y)})$$，即：    
    
$$    
\begin{align*}    
p & = \int_{-\infty}^{\infty}\frac{f(y)}{cg(y)}\times g(y)\text{d}y\\
  & = \frac{1}{c}\int_{-\infty}^{\infty}f(y)\text{d}y\\
  & = \frac{1}{c}    
\end{align*}    
$$   

因此，算法的迭代次数的期望即为$$E(N)=c$$。从这个角度上可以看出，若要使得算法的迭代次数尽可能少，等价于最小化上确界常数$$c$$。    

>算法成功采样一个随机变量$$X$$所需的迭代次数的期望即为上确界常数$$c=\text{sup}_x\{f(x)/g(x)\}$$。    

###<font color='#FF5544'>Rejection sampling算法证明</font>
由上述说明可知，要证明Rejection sampling算法可行，只需要证明$$\color{khaki}{\text{在给定条件}U\leqslant \frac{f(Y)}{cg(Y)}\text{下}，\text{随机变量}Y\text{的概率分布即为}F，\text{即}，P(Y\leqslant y|U\leqslant \frac{f(Y)}{cg(Y)})=F(y)}$$。    
令$$B=\{U\leqslant \frac{f(Y)}{cg(Y)}\}$$，$$A=\{Y\leqslant y\}$$，又因为$$P(B)=p=\frac{1}{c}$$，根据贝叶斯公式：    

$$
\begin{align*}
\frac{P(B|A)P(A)}{P(B)} &= P(U\leqslant \frac{f(Y)}{cg(Y)}|Y=y)\times \frac{G(y)}{1/c}\\
                        &= \frac{P(U\leqslant \frac{f(Y)}{cg(Y)}, Y=y)}{G(y)}\times \frac{G(y)}{1/c}\\
                        &= c\int_{-\infty}^yP(U\leqslant \frac{f(Y)}{cg(Y)}|Y=w\leqslant y)g(w)\text{d}w\\
                        &= c\int_{-\infty}^y\frac{f(w)}{cg(w)}g(w)\text{d}w\\
                        &= \int_{-\infty}^yf(w)\text{d}w\\
                        &= F(y)=P(A|B)=P(Y\leqslant y|U\leqslant \frac{f(Y)}{cg(Y)})~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\blacksquare
\end{align*}
$$


###<font color='#FF5544'>用Rejection sampling算法产生离散随机变量</font>    
产生离散随机变量的方法和产生连续随机变量的方法基本一致，只是这时候目标的概率分布函数$$F$$变成了概率质量函数$$p(k)=P(X=k)$$，辅助的概率分布$$G$$变成了概率质量函数$$q(k)=P(Y=k)$$，并满足条件$$\text{sup}_x\{p(k)/q(k)\}\leqslant c < \infty$$。

###<font color='#FF5544'>示例：产生服从简单Beta分布的随机变量</font>
Beta分布的概率密度函数为$$f(x)=bx^n(1-x)^m，x\in (0,1)$$，其中$$b=[\int_0^1x^n(1-x)^m\text{d}x]^{-1}$$为归一化常数。这里我们只研究$$n=m>1$$的情况，此时Beta分布的概率密度函数关于$$x=\frac{1}{2}$$对称。这里我们选择$$g(y)=1$$这一均匀分布作为辅助的概率密度函数。因此，$$c=\text{sup}_x\frac{f(x)}{g(x)}=\text{sup}_xf(x)$$当$$x=\frac{1}{2}$$时取到最大，即$$c=b4^{-n}$$。    
这样，若要产生服从$$f(x)=b(x(1-x))^n$$这一简单Beta分布的随机变量，只需要：    

    
<table border='2' cellpadding='15px' bordercolor='#F0E68C'>
<td>
<ol>
<li>产生两个0，1之间的均匀分布随机变量$U_1，U_2$。</li> <p/>
<li>If $U_2\leqslant 4^n(U_1(1-U_1))^n$，then $X=U_1$;<br/>
    Otherwise goto step 1。</li> 
</ol>
</td>
</table>


