# 【题解】[ABC299G] Minimum Permutation

## 题目链接

[[ABC299G] Minimum Permutation](https://www.luogu.com.cn/problem/AT_abc299_g)

## 题意概述

给定一个长度为 $N$ 的序列 $A$，由 $1$ 到 $M$ 之间的整数组成。其中，$1$ 到 $M$ 每个数至少出现一次。

找到一个长度为 $M$ 的 $A$ 的子序列，使得这个子序列中 $1$ 到 $M$ 恰好出现一次，输出满足条件的字典序最小的子序列。

## 数据范围

- $1\le M \le N \le 2\times 10^5$
- $1\le A_i \le M$
- $1$ 和 $M$ 之间的每个整数（包括 $1$ 和 $M$）在 $A$ 中至少出现一次。

## 题目分析

