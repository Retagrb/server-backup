# 【题解】[ABC312G] Avoid Straight Line

## 题目链接

[[ABC312G] Avoid Straight Line](https://atcoder.jp/contests/abc312/tasks/abc312_g)

## 题意概述

给定一棵 $n$ 个节点的树，第 $i$ 条边连接节点 $a_i$ 和 $b_i$，要求找到满足以下条件的三元整数组 $(i,j,k)$ 的数量：

- $1\le i<j<k\le n$；
- 对于树上任意一条简单路径，都不同时经过 $i,j,k$。

## 数据范围

- $1 \le n \le 2\times 10^5$
- $1\le a_i,b_i \le n$

## 思路分析

 题目要求对于树上任意一条简单路径都不同时经过 $i,j,k$，这个条件直接思考不容易想出来有什么处理的办法，考虑正难则反，我们可以先求出**树上至少有一条路径同时经过 $i,j,k$ 时满足条件的三元组 $(i,j,k)$ 的数量**，然后再容斥。

即，问题可以转化为：

> 树上所有满足 $1\le i<j< k\le n$ 的三元组 $(i,j,k)$ 的数量减去**树上至少有一条路径同时经过 $i,j,k$ 时，满足条件的三元组 $(i,j,k)$ 的数量**。

树上所有满足 $1\le i <j<k \le n$ 的三元组 $(i,j,k)$ 的数量实际上就是在 $1-n$ 的所有数中找到三个数 $(i,j,k)$ 使得 $1 \le i<j<k \le n$ 的数量，也就是 $\mathrm{C_n^3}=\dfrac{n\times (n-1)\times (n-2)}{6}$。

---

现在问题就只剩下：树上至少有一条路径同时经过 $i,j,k$ 时满足条件的三元组 $(i,j,k)$ 的数量怎么求。

有一条路径同时经过 $i,j,k$ 实际上就相当于是 $i$ 在 $j$ 到 $k$ 的路径上，或 $j$ 在 $i$ 到 $k$ 的路径上，或 $k$ 在 $i$ 到 $j$ 的路径上。

这就给了我们一个思路：我们首先可以钦定一个**中间节点 $u$**，即对于每一个点 $u$，其中 $1 \le u \le n$，满足有多少个不同于 $u$ 的**无序**点对 $(v,w)$ 使得 $u$ 在点 $v$ 到 $w$ 的路径上。这也就是当 $u$ 作为中间节点时符合条件的三元组的数量。

那么树上所有的点作为中间节点时的答案之和，就是树上至少有一条路径同时经过 $i,j,k$ 时满足条件的三元组 $(i,j,k)$ 的数量。

现在考虑怎么求当固定点 $u$ 作为中间节点时，有多少个不同于 $u$ 的无序点对 $(v,w)$ 使得 $u$ 在 $v$ 到 $w$ 的路径上。

考虑当 $u$ 作为整棵树的根时，显然要使得 $u$ 成为中间节点，那么当它的一个子树内的节点 $t$ 作为 $v$ 时，与 $t$ 在同一个子树内的节点一定不能作为 $w$，只有与它不在同一个子树内非 $u$ 的节点才有可能成为 $w$，也就是说其它子树内任意一个节点都可以作为 $w$。

也就是说对于 $u$ 的一个子树的根 $t$，该子树的点作为 $v$ 的方案数是 $sz_t\times (n-sz_t-1)$，由于是**无序**点对（即 $(v,w)$ 和 $(w,v)$ 算同一个点对，为了防止计算重复，那么我们需要在枚举 $u$ 的儿子时，每次减去当前已经被算过的点的个数，即我们可以定义一个 $now$，初始化 $now=n-1$，每次计算完一个子树 $t$ 的答案，就让 $now$ 减去 $sz_t$（因为这个子树内的点与后面子树的点的组合成为点对算过答案了，之后就没有必要再计算了），那么当前这个子树 $t$ 的答案 $ans_t=sz_t\times now$（具体详见下面的代码）。那么 $u$ 作为中间节点的点答案就是它所有的儿子的答案之和，即 $\sum \limits_{t\in son_u} ans_t$。

那么对于最后满足条件的三元组的数量 $ans$，我们只需要从 $1$ 开始 dfs 一遍，跑出所有点作为中间节点的答案然后加起来就可以了。

最终的答案就是 $\mathrm{C_n^3}-ans$。

## 代码实现

```cpp
//G
//The Way to Terminal Station…
#include<cstdio>
#include<iostream>
#include<cstring>
#define int long long
using namespace std;
const int maxn=2e5+10;
int sz[maxn];
int ans,n;

basic_string<int>edge[maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

void dfs(int x,int fa)
{
	int ret=0;
	for(int y:edge[x])
	{
		if(y==fa)continue;
		dfs(y,x);
		sz[x]+=sz[y];
	}
	sz[x]++;
	int now=n-1;
	for(int y:edge[x])
	{
		if(y==fa)continue;
		now-=sz[y];
		ret+=sz[y]*now;
	}
	if(edge[x].size()>1)ans+=ret;
	return ;
}

signed main()
{
	n=read();
	for(int i=1;i<n;i++)
	{
		int a,b;
		a=read();b=read();
		edge[a]+=b;
		edge[b]+=a;
	}
	int tot=n*(n-1)*(n-2)/6;
	dfs(1,0);
	cout<<tot-ans<<'\n';
	return 0;
}
```



