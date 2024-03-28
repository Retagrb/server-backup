# 【题解】P3092 [USACO13NOV]No Change G

状压 DP+双指针好题。

## 题目链接

[P3092 [USACO13NOV]No Change G](https://www.luogu.com.cn/problem/P3092)

## 题意概述

有 $n$ 个物品，每个物品的价格是 $a_i$。

现在要按顺序购买这 $n$ 个物品（即按照从 $a_1$ 到 $a_n$ 的顺序购买）。

有 $k$ 枚硬币，每一枚硬币的面值是 $w_i$，可以在购买这 $n$ 个物品过程中进行多次付款，每次付款最多支付一枚硬币，在任意时候，当前总共支付的硬币面值之和必须大于等于当前购买的物品价格之和。问在满足所有物品均能买到的前提下，能最多剩下多少面值的硬币。

## 数据范围

- $1\le n \le 10^5$
- $1 \le k \le 16$
- $1 \le w_i \le 10^8$
- $1 \le a_i \le 10^4$

## 思路分析

首先需要转化一步题意：将**最多剩下多少面值的硬币**转化为**所有硬币面值之和-买所有物品的最小代价**。

所有硬币面值之和这个就是 $\sum w_i$，我们只需要考虑如何求出买所有物品的最小代价。

这个数据范围一看就是状压。

比较显然的是随着购买物品的增多，其面值之和单调递增。

所以我们可以考虑定义 $dp_{sta}$ 表示在用了 $sta$ 这个状态下的硬币后，从左往右最远走到谁。

那么最后答案就是 $\sum \limits_{dp_{sta}=n}val_{sta}$。

这里的 $val_{sta}$ 表示的是选用 $sta$ 这个状态的硬币面值之和。这个可以直接预处理。

考虑如何求 $dp_{sta}$。

假设当前状态是 $now$，我们可以考虑在每次转移时只给当前状态上多选用一个硬币。

可以枚举下一个选用的硬币是谁，假设选用的是 $i$，那我们就要从 $dp_{now}$ 转移到 $dp_{now|(1<<i)}$ 上去。

考虑如何转移。

首先，对于下一个选用的硬币 $i$，所要支付的物品一定是从 $dp_{sta}+1$ 到该硬币能到达的最远的位置。换句话说，令 $l=dp_{sta}+1$，假设选用的区间是 $[l,r]$，那么这个 $r$ 一定是满足 $\sum \limits_{j=l}^k a_j \le w_i$ 中最大的 $k$。

考虑如何求出这个最大的 $k$。

我们可以预处理出一个数组 $f_{i,l}$ 表示选用第 $i$ 个硬币从 $l$ 开始时能够支付的最远的物品位置。

以样例为例，当 $i=2,l=1$ 时，$f_{i,l}=4$，因为从 $1$ 到 $4$ 的所有 $\sum a_i=14<15$，再加一个就是 $17>15$ 了。所以最远能到达 $4$。

可以发现对于每一个 $i$，这个 $f$ 数组是满足单调性的，所以我们可以双指针从左向右跑一遍。单次时间复杂度 $O(n)$，这部分总时间复杂度 $O(nk)$。

那么对于 $dp_{now|(1<<i)}$，有：
$$
dp_{now|(1<<i)}=\max(dp_{now|(1<<i)},f[i][dp_{now}+1])
$$
那么我们的转移就完成了。

总时间复杂度 $O(nk+2^k k)$。

## 代码实现

```cpp
//luoguP3092
//2022.9.22 22:02
#include<cstdio>
#include<iostream>
//#define debug1
//#define debug2
//#define debug3
//#define debug4
#define int long long
using namespace std;
const int maxk=20;
const int maxn=1e5+10;
int dp[1<<maxk],w[maxk],a[maxn];
int val[1<<maxk],f[maxk][maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

signed main()
{
	int k=read(),n=read(),tot=0;
	for(int i=1;i<=k;i++)w[i]=read(),tot+=w[i];
	for(int i=1;i<=n;i++)a[i]=read();
	for(int sta=0;sta<(1<<k);sta++)
	{
		for(int i=0;i<k;i++)
		{
			if(sta&(1<<i))val[sta]+=w[i+1];
		}
	}
	int ans=1e18;
	for(int sta=0;sta<(1<<k);sta++)
	{
		for(int i=0;i<k;i++)
		{
			if(sta&(1<<i))continue;
			if(dp[sta]>=n)continue;
			dp[sta|(1<<i)]=max(dp[sta|(1<<i)],f[i+1][dp[sta]+1]);
		}
		if(dp[sta]==n)ans=min(ans,val[sta]);
	}
	if(ans==1e18)cout<<-1<<'\n';
	else cout<<tot-ans<<'\n';
	return 0;
}
```



