---
title: Tarski 问题
date: 2024-02-28 20:52:50
tags: [计算理论,离散数学,论文导读]
mathjax: true
---
作者应该是俩中国人

# Tarski定理
对于完全格 $<L,\preceq>$,以及$L \to L$的单调增函数$f$，$f$一定存在不动点

有点像格上的介值定理
# 文章贡献
对于n阶k维格点格(grid lattice),以及定义在其上的单调函数$f$,构造出了一个$O(\log^{\lceil \frac{k+1}{2} \rceil}n)$算法找到一个不动点。
# 算法流程
在k+1个维度中选择一个切片，比如$x_{k+1}=\lceil\frac{n}{2}\rceil$,在这个切片上，一定有一个点要么是前驱点要么是后缀点，（前驱点指$x \preceq f(x)$,后缀点同理）

（证明很简单，指考虑前$k$维，一定存在一个不动点，即$\exist x$ $st.$ $\forall i \in[k],x_i=f(x)_i$，那么$x$就一定是一个前驱or后缀点)

假设找到前驱点$l$的复杂度为$q(n,k)$,此时，在$l$到上界$r=[n]^k$的子格中，$f$一定封闭，由于$f$单调且$ranf\in L_{l,r}$,根据Tarski定理，$L_{l,r}$中一定存在一个不动点,因此问题规模缩减一半，总复杂度的上界为$O(2^k+k\log n \cdot q(n,k))$,因此，问题变成了如何在切片上求prefix/suffix

为了在切片上求prefix/suffix，作者引入了一个新问题记作Tarski*问题：

## Tarski*(n,k)
$Tarski*(n,k)$是指如下问题：

定义函数$g,[n]^k \to {-1,0,1}^{k+1}$,为:$\forall i \in [k],g(x)_i=sgn(f(x)_i-x_i),g(x)_{k+1}=sgn(f(x)_{k+1}-\lceil \frac{n}{2} \rceil)$,也就是说，$g$是表示一个点的偏移的函数，Tarski*问题就是找到一个$x$使得${g(x)_i}$中不同时存在$1,-1$

很显然这样的$x$就是前驱or后继点。

很可惜，这样并没有简化问题，因此我们再引入refined Tarski*(n,k)问题.

## Refined Tarski*(n,k)
对于满足如下条件的函数$g,[n]^k \to \{-1,0,1\}^{k+1}$:

1. $\forall i \in [k] and x \in L,x_i+g(x)_i \in [n]$
2. $(x,0)+g(x)$是单调不降函数

可以计算出$p^l,p^r$,满足:$\forall i \in [k] g(p^l)_i \geq 0 ,g(p^r)_i \leq 0$,并且$g(p^l)_{k+1}=g(p^r)_{k+1}$

## Refined Tarski*(n,k) 问题的求解
好像很简单，暂时懒得写了，留作习题

# 核心定理以及其证明
这篇文章的精髓，或者说是解决Tarski不动点问题的关键是如下定理:

假设，Tarski*(n,a) 可以在 $q(n,a)$复杂度内求解，Tarski*(n,b) 可以在 $q(n,b)$复杂度内求解，那么 Tarski*(n,a+b)可以在$O((b+1)q(n,a)q(n,b))$内求解

## 算法
注意，下述算法中 Tarski*(n,b)依赖的$g$函数是在线生成的，你可能会担心这样的$g$不满足Tarski*问题对$g$函数的限制，但是请不要惊慌，因为最终我们会证明这样的$g$是满足1. $\forall i \in [k] and x \in L,x_i+g(x)_i \in [n]$
2. $(x,0)+g(x)$是单调不降函数 的。

```cpp
q=[];//前几轮 Tarski*(n,b)的询问记录
r=[];//前几轮对 q_i的回答
pl=[];
pr=[];
for(i=1;;++i){
    pl[i]=[1]^k;
    pr[i]=[n]^k;
    q[i]=Tarski*(n,b).query;
    for(j=1;j<i;++j){
        if(q[j]<=q[i])pl[i]=max(pl[i],pl[j]);
        if(q[i]<=q[j])pr[i]=min(pr[i],pr[j]);
    }
    for(j=a+1;j<=a+b+1;++j){
        构造g_j函数为 g(x,q[i])的前a维和第j维;
        用g_j计算在[pl[i],pr[i]] 内refined Tarski*(n,a)的解，并更新pl[i],pr[i]
    }
    for(t=a+1;t<=a+b+1;++t){
        r[i][t-a]=g(pl[i],q[i])[t];
    }
    若r[i]全非负 return (p[l][i],r[i]);
    若r[i]全非正 return (p[r][i],r[i]);
}
```
可以注意到如下事实:

1. 由于refined Tarski* 问题的特性，最后更新出来的 $p^l_i,p^r_i$一定满足前a维度l正r负，其它维度lr都一样.

2. 对Tarski*(n,b)的回复是满足条件的：注意到在第一个小for里，我们保证了$p^l_i$一定比所以更小询问的$p^l$大，因而$q^{x}_j+r^{x}_j$的每一项都比$q^{i}_j+r^{i}_j$小

3. 由 1.可知，最后如果回复满足Tarski*(n,b)的要求，那么我们就求出了Tarski*(n,a+b)

# 最后步骤以及总结
在另一篇文章中提到了，Tarski*(n,2) 可以 $O(\log n)$求解，因此，按照刚刚证明的定理， Tarski*(n,k) 可以$O(\log^{\lfloor \frac{k}{2} \rfloor}n)$求解 ，因此 Tarski(n,k)可以$O(\log^{\lceil \frac{k+1}{2} \rceil}n)$求解