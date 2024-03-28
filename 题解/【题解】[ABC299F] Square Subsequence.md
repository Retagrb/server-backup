# 【题解】[ABC299F] Square Subsequence

## 题目链接

[[ABC299F] Square Subsequence](https://www.luogu.com.cn/problem/AT_abc299_f)

## 题意概述

给定一个由小写英文字母组成的字符串 $S$。计算满足以下条件的非空字符串 $T$ 的数量，答案对 $998244353$ 取模。

> 将 $T$ 复制一倍形成 $TT$，则 $TT$ 是 $S$ 的子序列（不一定连续）。

## 数据范围

- $1\le |S|\le 100$

## 题目分析

一看这个数据范围，大概率是 $O(n^4)$ 或者是 $O(n^3 \log n)$ 的。

题目要求的是一个满足条件的 $TT$ 成为 $S$ 的子序列时 $T$ 的数量。

那么我们首先应该解决一个问题：

> 假如 $T$ 是已知的，那么我们应该如何从 $S$ 中提取子序列 $TT=T_1,T_2,T_3,\cdots,T_n,T_1,T_2,T_3,\cdots,T_n$。

由样例 $2$ 可知，从 $S$ 中提取字符串可能有多种方式，因此我们需要避免多次计数相同的子序列。我们可以通过**尽可能提取 $S$ 的左侧字符**的原则来避免重复计数。

具体做法：

定义 $s(i,c)$ 表示小写英文字母 $c$ 在第 $i$ 个及之后的字符 $S_{i+1}, S_{i+2}, ..., S_{N}$ 中出现的最左位置。

特殊地，如果第 $i$ 个及之后字符中不包含 $c$，则我们将 $s(i,c)$ 定义为 $∞$。

那么我们可以通过如下步骤在 $S$ 中提取 $TT$，首先考虑第一个 $T$：

- 提取 $S$ 的第 $p_1$ 个字符，其中 $p_1=s(0,T_1)$。
- 提取 $S$ 的第 $p_2$ 个字符，其中 $p_2=s(p_1,T_2)$。
- 提取 $S$ 的第 $p_3$ 个字符，其中 $p_3=s(p_2,T_3)$。 
- $\cdots$
- 提取 $S$ 的第 $p_n$ 个字符，其中 $p_n=s(p_{n-1},T_n)$。

再考虑第二个 $T$：

- 提取 $S$ 的第 $q_1$ 个字符，其中 $q_1=s(p_n,T_1)$。
- 提取 $S$ 的第 $q_2$ 个字符，其中 $q_2=s(q_1,T_2)$。
- 提取 $S$ 的第 $q_3$ 个字符，其中 $q_3=s(q_2,T_3)$。
- $\cdots$
- 提取 $S$ 的第 $q_n$ 个字符，其中 $q_n=s(q_{n-1},T_n)$。

这样我们就唯一地确定了从每个 $S$ 的子串提取子序列的方法，因此每个子序列 $T$ 都可以计算而没有重复计数。

这个过程可以线性预处理完成，时间复杂度 $O(26n)$。

---

这是当在已知 $T$ 的情况下，我们如何从 $S$ 中提取 $TT$。

那么我们在不知道 $T$ 的情况下，如何统计满足条件的 $T$ 的数量呢？

我们首先可以枚举 $S$ 中第二个 $T_1$ 出现在哪里，即 $q_1$。

对于一个固定的 $q_1=x$，我们从 $p_1=s(0,T_1)=s(0,S_x)$ 的位置开始，按照上述步骤依次确定 $TT$ 的第 $2$ 个及之后的字符，同时从 $q_1=x$ 的位置开始按照**尽可能提取 $S$ 的左侧字符**的原则提取 $TT$ 的第二个 $T$ 部分。具体来说：

- 首先枚举 $T$ 的第二个字符 $T_2$ 是哪个小写英文字母。设 $p_2=s(p_1,T_2)$，$q_2=s(q_1,T_2)$，然后提取 $S$ 的 $p_2$ 和 $q_2$ 位置的字符。

- 然后枚举 $T_3$ 是哪个小写英文字母。设 $p_3=s(p_2,T_3)$，$q_3=s(q_2,T_3)$，然后提取 $S$ 的 $p_3$ 和 $q_3$ 位置的字符。

- 以此类推，直到枚举 $T_n$ 是哪个小写英文字母。设 $p_n=s(p_{n-1},T_n)$，$q_n=s(q_{n-1},T_n)$，然后抽取 $S$ 的 $p_n$ 和 $q_n$ 位置的字符。

那么我们要计算 $q_1=x$ 的时候的答案，就可以通过 DP 来解决。

定义 $dp_{i,j}$ 表示的是满足 $p_n=i$ 且 $q_n=j$ 的 $T$ 的个数。

转移：

$$
dp_{s(i,c),s(j,c)}←dp_{s(i,c),s(j,c)}+dp_{i,j}
$$

那么对于固定的 $q_1=x$ 答案就是 $\sum_{s(i,S_x)=x}\sum_j dp_{i,j}$

总答案就是所有可能的 $q_1$ 的答案之和。

DP 复杂度为 $O(26\times n^2)$，枚举 $q_1$ 的复杂度是 $O(n)$，所以总复杂度为 $O(26\times n^3)$。

## 代码实现

```cpp
//F
#include<cstdio>
#include<iostream>
#define int long long 
using namespace std;
const int maxn=105;
const int mod=998244353;
int dp[maxn][maxn],s[maxn][26];
//dp[i][j] 表示的是满足 p_n=i 且 q_n=j 的 T 的个数。
//s[i][c] 表示的是 c 在第 i 个位置后出现的第一个位置。

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

signed main()
{
	string S;
	cin>>S;
	int n=S.size();
	S='%'+S;
	//预处理 s 数组。
	for(int i=0;i<26;i++)s[n][i]=n+1;
	for(int i=n-1;i>=0;i--)
	{
		for(int j=0;j<26;j++)s[i][j]=s[i+1][j];
		s[i][S[i+1]-'a']=i+1;
	}
	int ans=0;
    //dp 部分
	for(int q=2;q<=n;q++)//枚举 q_1
	{
		int p=s[0][S[q]-'a'];//p 就是 p_1
		if(p>=q)continue;
         //初始化
		for(int i=1;i<=n;i++)
		{
			for(int j=1;j<=n;j++)dp[i][j]=0;
		}
		dp[p][q]=1;
         //dp 转移
		for(int i=1;i<=n;i++)
		{
			for(int j=1;j<=n;j++)
			{
				for(int k=0;k<26;k++)
				{
					int nxti=s[i][k],nxtj=s[j][k];
					if(nxti>=q||nxtj>n)continue;
					(dp[nxti][nxtj]+=dp[i][j])%=mod;
				}
			}
		}
		for(int i=1;i<=n;i++)
		{
			for(int j=1;j<=n;j++)
			{
				if(s[i][S[q]-'a']==q)(ans+=dp[i][j])%=mod;
			}
		}
	}
	cout<<ans<<'\n';
	return 0;
}
```
