# 【题解】P6880 [JOI 2020 Final] オリンピックバス

## 题目链接

[P6880 [JOI 2020 Final] オリンピックバス](https://www.luogu.com.cn/problem/P6880)

## 题意概述

给定一个 $N$ 点 $M$ 边的有向图，每条边从 $U_i$ 指向 $V_i$，经过这条边的代价为 $C_i$。

点编号为 $1$ 到 $N$。

我们可以翻转一条边，即让他从 $U_i$ 指向 $V_i$ 变为从 $V_i$ 指向 $U_i$，这时会有 $D_i$ 的代价产生。

你要从点 $1$ 到点 $N$，再从点 $N$ 回到点 $1$，你想知道，通过翻转一条边，或者不翻转，能得到的最小代价和为多少？

## 思路分析

这个题一个误导的地方就是很容易想到分层图上去。

