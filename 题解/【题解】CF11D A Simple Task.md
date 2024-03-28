# 【题解】CF11D A Simple Task

## 题目链接

[CF11D A Simple Task](https://www.luogu.com.cn/problem/CF11D)

## 题意概述

给定一张 $n$ 个点 $m$ 条边的无向图，无重边自环，点数不超过 $19$，求无向图中环的数量。

## 思路分析

看到数据范围，考虑对点数状压。

一个比较常见的套路是将一个环拆成两个点之间的路径，即对于环上两点 $i,j$，环的长度等于 $dis_{i,j}+dis_{j,i}$，即一条从 $i$ 到 $j$ 的路径加上另一条从 $j$ 到 $i$ 的路径。

那么我们就可以把环转化成路径，即用 $dp_{i,j,sta}$ 表示状态 $sta$ 中从 $i$ 开始到 $j$ 结束的简单路径条数。

那么最后的答案就为对于每个 $i$ 而言所有的 $dp_{i,i,sta}$ 之和。

考虑空间：$19 \times 19 \times 2^{19}=189267968$，此题空间限制是 `250mb`，显然会爆掉。考虑优化空间。

状压中比较常见的一种优化办法就是用 $lowbit$ 来优化。

我们每次考虑只让一个状态中最小的点，即 $lowbit(sta)$ 作为起点，这样做相当于是给它规定了一个顺序，可以做到不重不漏。

那么我们可以定义 $dp_{sta,i}$ 表示状态 $sta$ 从 $lowbit(sta)$ 开始到 $i$ 结束的简单路径条数。

考虑转移。

首先枚举集合 $sta$ 中的每一个元素 $i$，再枚举所有与之连边的 $j$ 作为下一个要加入集合的点。

那么 $j$ 就需要分类讨论了：

- 当 $j$ 在集合 $sta$ 内时：

	- 当 $j= lowbit(sta)$ 时，这是一个特殊情况。也就是说从 $i \to j,j  \to i$ 构成了一个回路（环），所以将此时的 $dp_{sta,i}$ 累加入答案即可。
	- 当 $j \ne lowbit_{sta}$ 时，由于我们的 $j$ 是枚举的下一个要加入集合的点。所以这里不做处理。

- 当 $j$ 不在集合 $sta$ 内时：

	将 $j$ 加入集合，那么有：
	$$
	dp_{sta|(1<<j),j}=dp_{sta|(1<<j),j}+dp_{sta,i}
	$$

直接转移即可。

还有一个易错点：

对于每一个算出来的答案，我们会将无向图产生的**重边**计算成二元环，所以答案要减去 $m$。并且会将所有点数大于 $3$ 的环多计算一次（顺时针一次，逆时针一次），所以答案要除以 $2$。

## 障碍点

- 没有考虑到空间产生的问题，从而没有想到利用 $lowbit$ 来优化。

	因此以后每道题目还是想到一个思路后先计算一下空间，防止 `MLE`。

## 易错点：

- 答案应为 $(ans-m)/2$ 而非 $ans$，原因在上面思路分析中已经说明过。

## 一些技巧

此题包含两个重要技巧：

- 对于环的问题应该想到将其拆分成环上两点 $i \to j$ 和 $j \to i$ 的两条路径；
- 对于状压要考虑到 $lowbit$ 优化。

## 代码实现

```cpp
//CF11D
#include<cstdio>
#include<iostream>
#define int long long
using namespace std;
const int maxn=20;
int dp[1<<maxn][maxn],a[maxn][maxn];
int ans=0;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int lowbit(int x){return x&(-x);}

signed main()
{
	int n=read(),m=read();
	for(int i=1;i<=m;i++)
	{
		int u,v;
		u=read();v=read();
		u--;v--;
		a[u][v]=a[v][u]=1;
	}
	for(int i=0;i<n;i++)dp[1<<i][i]=1;
	for(int sta=1;sta<(1<<n);sta++)
	{
		for(int i=0;i<n;i++)
		{
			if(!dp[sta][i])continue;
			if(!(sta&(1<<i)))continue;
			for(int j=0;j<n;j++)
			{
				if(!a[i][j])continue;
				if(lowbit(sta)>(1<<j))continue;
				if(lowbit(sta)==(1<<j))ans+=dp[sta][i];
				else if(!(sta&(1<<j)))dp[sta|(1<<j)][j]+=dp[sta][i];
			}
		}
	}
	cout<<((ans-m)>>1)<<'\n';
	return 0;
}
```

