---
layout: post
title: "Why Modulo $1e9+7$"
date: 2014-11-09 23:07
comments: true
categories: [Number Theory]
---

一直很好奇很多编程问题，比如求大数阶乘，大数的排列组合等，要求将输出结果对$$1e9+7$$(=1000000007)取模。为什么是这样一个数呢？今天一查，网上还是有不少讨论的。总结一下有下面几个原因。

*	首先因为$$1e9+7$$是一个质数
*	其次是$$1e9+7$$对于int32来说足够大
*	还有就是$$1e9+7$$的平方对于int64来说也恰好不会溢出

<!--more-->
其实对于后两点来说都是比较好理解的。足够大是因为取模的结果尽量不重复，跟哈希表一样(应该说哈希表跟它一样，明显它更本质一点#_#)。对于平方不会对int64溢出意味着对于很大数相乘依然可以利用$$1e9+7$$取模到int32基本数据类型中。而对于为什么要求质数，等介绍完模运算就能很好的理解了。

##模运算
假设有$$a\equiv c(\text{mod} ~m)$$， $$b\equiv d(\text{mod} ~m)$$，则对于基本算术运算有：

*	$a+b\equiv	c+d(\text{mod} ~m)$
*	$a-b\equiv	c-d(\text{mod} ~m)$
*	$a\times b\equiv c\times d(\text{mod} ~m)$
*	$a/ b\not\equiv c/ d(\text{mod} ~m)$

而且满足以下分配率：

*	$( a + b ) \% c = ( ( a \% c ) + ( b \% c ) ) \% c$
*	$( a * b ) \% c = ( ( a \% c ) * ( b \% c ) ) \% c$
*	$( a – b ) \% c = ( ( a \% c ) – ( b \% c ) ) \% c$
*	$( a / b ) \% c \neq ( ( a \% c ) / ( b \% c ) ) \% c$

注意到取模除法是不满足分配率的。(万恶的根源。。。)

对于上述分配率一个很常见的应用就是求大数的阶乘了，结果对$$M=1e9+7$$取模。

```c++
typedef long long ll;
ll factorial(int n, int M) {
    ll ans = 1;
    while(n >= 1) {
        ans = (ans * n);
        n--;
    }
    return ans % M;
}
```
如果直接计算对最后的结果再取模的话很可能在累乘的时候就已经溢出int64了，所以这种写法是错误的。正确的写法就是利用模乘法的分配率的性质在每一步乘法计算的时候都对结果取模。

```c++
typedef long long ll;
ll factorial(int n, int M) {
    ll ans = 1;
    while(n >= 1) {
        ans = (ans * n) % M;
        n--;
    }
    return ans;
}
```

下面就来详细说说为什么一定要$$M$$是一个质数。首先还得从模除法不满足分配率开始说起。
既然除法不满足分配率，那把除法转化成乘法不就行了。那么模除法能不能像一般数值运算一样简单的把除以
$$a$$变成乘以$$a^{-1}$$呢。直观上想好像不行，但是有什么可以难倒代数学家呢，人家早就从群啊环啊什么的定义好这些了，直接搬过来就行了，那就是——$$\ulcorner$$ **逆元** $$\lrcorner$$。

##逆元

逆元的定义很简单：满足$$ax\equiv 1(\text{mod} ~M)$$，则$$x=a^{-1}$$就是$$a$$的逆元。

一旦有了逆元，$$(a / b) \% c$$就可以写成$$(a * b^{-1}) \% c$$。这样转换成乘法之后分配率也适用了。

但是注意到如果$$\text{gcd}(a,M)!=1$$的话，$$a$$的逆元是不存在的。所以如果$$M$$是一个质数的话$$\text{gcd}(a,M)!=1$$一般都成立了吧。这还不够！因为如果$$a$$恰好是$$M$$的倍数的话依然不满足条件啊，所以如果$$M$$足够大的话，比如$$1e9+7$$([一般问题也不会让你求$$1e9+7$$的阶乘吧](http://www.bilibili.com/video/av362069/))，基本就能够满足$$\text{gcd}(a,M)!=1$$这个条件了。

那么如何计算一个数的逆元呢？

最简单的就是暴力搜索了。对于$$ax\equiv 1(\text{mod} ~M)$$只需要对$$x=[1, M-1]$$(因为对$$M$$取模的结果是以$$0,1,2,\ldots,M-1$$为周期的)计算一遍找出和$$a$$相乘$$\text{mod} ~M$$等于1的数就是$$a$$的逆元了。但是这种算法的复杂度是$$\mathcal{O}(M)$$。而一般情况下$$M$$的取值非常大，如本文讨论的$$1e9+7$$这么大的话，这种线性复杂度就是不可以接受的了。更高效的算法有两种，一种是
**Extended Eucledian algorithm** (扩展欧几里德算法)，另外一种是利用**Fermat's Little Theorem** (费马小定理)。

##Extended Eucledian algorithm
我们都知道欧几里德算法，又叫辗转相除法是用来求两个数的最大公约数的。那么扩展欧几里德算法又是干什么的呢。扩展欧几里德算法是用来求解不定方程：$$ax+by=\text{gcd}(a,b)$$(这个方程又叫[贝祖等式](http://en.wikipedia.org/wiki/B%C3%A9zout%27s_identity))。

求满足$$ax\equiv 1(\text{mod} ~M)$$，$$a$$的逆元只需要将方程变形成$$ax-My=1$$，因为要满足$$\text{gcd}(a,M)=1$$，所以等式右边为$$1$$。这样就可以利用扩展欧几里德算法来解出$$x$$。

<table border='2' cellpadding='15px' bordercolor='#FF5544'>
<td>
<ol>
<li>当$b=0$时，$\text{gcd}(a,b)=a$。此时$x=1$，$y=0$。(递归终止条件)</li> <p/>
<li>当$ab!=0$时，设原始方程为：$ax_1+by_1=\text{gcd}(a,b)$，并存在：
   $bx_2+(a\%b)y_2=\text{gcd}(b,a\%b)$。由辗转相除法可知$\text{gcd}(a,b)=\text{gcd}(b,a\%b)$<br/>
   所以有：$ax_1+by_1=bx_2+(a\%b)y_2$。<br/>
   即：$ax_1+by_1=bx_2+(a-(a/b)\times b)y_2=ay_2+b(x_2-(a/b)\times y_2)$<br/>
   所以：$x_1=y_2$$，$$y1=x2-(a/b)\times y_2$<br/>
   也就是说只要求出$x_2$和$y_2$就能根据上述等式求得$x_1$和$y_2$。这样就可以利用递归定义，直到满足递归终止条件为止，再一步步返回计算出$x$和$y$。</li> <p/>
</ol>
</td>
</table>
<br/>


```c++
int extgcd(int a, int b, int &x, int &y) {
    int d = a;
    if (b != 0) {
    	d = extgcd(b, a % b, y, x);
        y -= (a / b) * x;
    } else {
    	x = 1;
        y = 0;
    }
    return d;
}
```

扩展欧几里德算法的复杂度和辗转相除法是一样的(只是多了一步计算)，都是
$$\mathcal{O}(\text{log}~\text{max}(a,b))$$。利用扩展欧几里德算法就可以很方便地求得
$$a$$的逆元($$\text{mod} ~M$$)。

```c++
int mod_inverse(int a, int M) {
    int x, y;
    extgcd(a, M, x, y);
    return (M + x % M) % M;
}
```
这里用不到extgcd的返回值了(因为肯定是$$1$$)。返回值的时候加上$$M$$是为了防止当$$x$$为负数的时候，有些编译器会将$$x\%M$$返回负数，加上$$M$$让其值回到$$[0,M-1]$$。

##Fermat's Little Theorem
费马小定理说的是：对于任意正整数$$a$$和质数$$p$$，满足$$a^p\equiv a(\text{mod} ~p)$$。而且如果$$a$$不是$$p$$的倍数时，条件等价成$$a^{p-1}\equiv 1(\text{mod} ~p)$$。利用这条性质可以很方便的求解逆元。$$ax\equiv 1(\text{mod} ~M)$$，这里质数$$p=M$$，则$$a^{p-1}\equiv 1(\text{mod} ~p)$$=$$a\cdot a^{M-2}\equiv 1(\text{mod} ~M)$$。也就是说$$a(\text{mod} ~M)$$的逆元$$x=a^{M-2}(\text{mod} ~M)$$。这样就可以通过快速幂运算来求解逆元。

```c++
typedef long long ll;
ll fast_pow(ll a, ll n, ll M) {
    ll res = 1;
    while (n > 0) {
    	if (n & 1) res = res * a % M;
        a = a * a % M;
        n >>= 1;
    }
    return res;
}

ll mod_inverse(ll a, ll M) {
    return fast_pow(a,M-2,M);
}
```
说了这么多，总结一下就是在一些组合问题中由于数值太大需要对结果取模。在计算的时候，特别是在计算排列组合的时候会不可避免的遇到模除法，为了使得除法计算起来符合数值运算的过程就需要引入逆元的概念，在计算逆元的时候$$M$$一定要是一个质数，这个质数一定得足够大，要不然取模后的结果很多都相同了造成无法判断问题输出的正确性。所以很多问题就用了$$1e9+7$$这个数了。。。。