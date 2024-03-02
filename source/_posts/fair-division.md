---
title: fair division
date: 2024-02-28 17:59:11
tags: [计算理论,组合数学,unsolved]
---
尝试了一个寒假的open problem，先放一段时间再想吧。。

先记录一下目前的思路们:
# 1
先用$d_1$视角构造一组allocation $X=(X_1,X_2,X_3)$ 满足对 $d_1$ tEFX，不失一般性，令 $X_3 <_{3} X_1 <_{3} X_2$
将 $X_1$中的chores 按照 $d_3$ 升序排序，即 $X_1={c_1,c_2....},d_3(c_1)<d_3(c_2)...$
然后执行

```
for i=1 to |X_1|{
    if(d_3(X_1-c_i)<d_3(X_3+c_i))break
    X_3.add(c_i)
    X_1.delete(c_i)
}
```
此时，必然有 $d_3(X_3)<d_3(X_1)<d_3(X_2)$,且 $X_3,X_1$都对agent3 tEFX-feasible. 同时，$d_1(X_1)<d_1(X_3)$.

如果 $X_2$ is tEFX feasible for agent1,那么算法结束

反之，$X_2$ is not tEFX feasible for agent1, 则一定有$d_1(X_2)>d_1(X_1)$，此时以$d_1$为disutility function使用PR算法,再回到算法开始。
## 缺陷
无法保证 $min(d_1(X_1),d_1(X_2))$单调,无法确定算法是否有穷
已找到一组反例，在$X_1,X_3$之间来回传递。

```
d[1][1]=12;d[1][2]=8;d[1][3]=6;d[1][4]=1;d[1][5]=17;
d[2][1]=7;d[2][2]=1;d[2][3]=11;d[2][4]=2;d[2][5]=1;
d[3][1]=17;d[3][2]=6;d[3][3]=14;d[3][4]=4;d[3][5]=7;
```

# 2
先用$d_1$视角构造一组allocation $X=(X_1,X_2,X_3)$ 满足对 $d_1$ tEFX，不失一般性，令 $X_3 <_{3} X_1 <_{3} X_2$
将 $X_1$中的chores 按照 $d_3$ 升序排序，即 $X_1={c_1,c_2....},d_3(c_1)<d_3(c_2)...$

如果出现 $X_1,X_2$都对agent2，3 不 tEFX feasible, 那么 此时直接对agent2或agent3的disutilities function使用PR算法,在开局就重构

## 缺陷：
也是没有单调性

# 3
直接摈弃原来的思路，直接对disutilities function的值域归纳。
也即，假设一组disutilities function可以做出tEFX，只要证明将其中任意一个值增加1仍然可以做出tEFX分配。

正在讨论。。也是一堆问题

upd:讨论不动了，研究值域是没有未来的