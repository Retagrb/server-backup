# 【题解】[ABC318G] Typical Path Problem

## 题目链接

[G - Typical Path Problem](https://atcoder.jp/contests/abc318/tasks/abc318_g)

## 题意概述

给定一个 $n$ 个点 $m$ 条边的无向连通图。

给定三个该图上的不同顶点 $A,B,C$，问是否存在一条从 $A$ 到 $C$ 的简单路径，使得 $B$ 经过这条简单路径。

## 数据范围

- $3 \le n \le 3\times 10^5$
- $1 \le A,B,C \le n$

## 思路分析

[P4630 铁人两项](https://www.luogu.com.cn/problem/P4630)的弱化版。

涉及到无向图的简单路径，想到 Tarjan 缩点。本题涉及到的是点连通性的问题，考虑到将原图进行点双缩点变成圆方树，在圆方树上求解。

圆方树有一个性质：

> 对于广义圆方树上一条从 $u$ 到 $v$ 的简单路径，设为 $u \to s_1 \to c_1 \to s_2 \to c_2 \to \cdots \to c_k \to s_{k + 1} \to v$，其中 $s$ 为方点，$c$ 为圆点。
>
> 那么有：$u \rightsquigarrow v$ 所有路径上点的并集恰好是 $s_1, s_2, s_3, \ldots s_{k + 1}$ 的并集。

由于点双的缩点后的圆方树上，每个方点 $s$ 对应的是点双，圆点 $c$ 对应的是原图中的点，且对于每一个方点，它对应的点双向这个点双连通分量中的每个点连边。那么根据上述性质，对于给定的点 $A,B,C$，若 $B$ 在 $A$ 到 $C$ 的路径上，当且仅当 $B$ 与 $s_1,s_2,s_3,\cdots,s_{k+1}$ 中的一个或几个方点相连，也就是说，$B$ 必须是 $s_1,s_2,s_3,\cdots,s_{k+1}$ 中一个或几个所表示的点双中的点。

那么我们考虑预处理 $B$ 所在点双连通分量（这个可以在缩点的时候完成），然后判断它所在的每一个连通分量是否在圆方树上 $u\rightsquigarrow v$ 的路径上。

> 注意：在点双连通分量中，一个点可能在多个点双连通分量里。所以需要预处理 $B$ 所在的**所有**点双连通分量。

---

现在问题转化为：

> 如何判断一棵树上，一个点 $w$ 是否在 $u$ 到 $v$ 的路径上。

有一个比较套路的方法是：找到 $u$ 和 $v$ 的 LCA，然后分别判断 $w$ 是否在 $u$ 到 LCA 的路径上和 $v$ 到 LCA 的路径上，这两个只要有一个满足就说明 $w$ 在 $u$ 到 $v$ 的路径上。

那么现在问题又转化成如何判断 $w$ 在 $u$ 到 LCA 的路径上。

我们令点 $p$ 是 $u$ 和 $v$ 的 LCA。那么首先就是 $w$ 的深度 $dep_{w}$ 必须在区间 $[dep_{p},dep_{u})$ 内，然后我们可以从 $u$ 往上倍增的跳，跳到 $w$ 的深度，如果当前这个点刚好是 $w$，则 $w$ 在 $u$ 到 $p$ 的路径上；反之，不在。

---

这样这个问题就完美解决了。

梳理一下，这道题的求解步骤是：首先点双缩点得到圆方树，然后判断 $B$ 所在的点双是否在圆方树上 $A$ 到 $C$ 的路径上，转化为 $A$ 到 LCA 的路径和 $C$ 到 LCA 的路径上即可。

## 代码实现

```cpp
//G
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<cstring>
#include<vector>
using namespace std;
const int maxn=2e5+10;
int dfn[maxn],low[maxn],val[maxn],tot,num;
int stk[maxn],vis[maxn],k[maxn],sz,rt,cnt;
int fa[maxn<<1][25],dep[maxn<<1];

basic_string<int>edge[maxn],edge2[maxn<<1];

vector<int>dcc[maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

void dfs(int x)
{
	dfn[x]=low[x]=++tot;
	stk[++sz]=x;
	num++;
	for(int y:edge[x])
	{
		if(!dfn[y])
		{
			dfs(y);
			low[x]=min(low[x],low[y]);
			if(low[y]==dfn[x])
			{
				cnt++;
				edge2[x]+=cnt;
				edge2[cnt]+=x;
				dcc[x].push_back(cnt);
				val[cnt]++;
				while(1)
				{
					edge2[stk[sz]]+=cnt;
					edge2[cnt]+=stk[sz];
					dcc[stk[sz]].push_back(cnt);
					val[cnt]++;
					if(stk[sz--]==y)break;
				}
			}
		}
		else low[x]=min(low[x],dfn[y]);
	}
}

void dfs2(int x,int fath)
{
	fa[x][0]=fath;
	for(int i=1;i<=20;i++)fa[x][i]=fa[fa[x][i-1]][i-1];
	for(int y:edge2[x])
	{
		if(y==fath)continue;
		dep[y]=dep[x]+1;
		dfs2(y,x);
	}
	return ;
}

int lca(int u,int v)
{
	if(dep[u]<dep[v])swap(u,v);
	for(int i=0;i<=20;i++)
	{
		if((dep[u]-dep[v])&(1<<i))u=fa[u][i];
	}
	if(u==v)return u;
	for(int i=20;i>=0;i--)
	{
		if(fa[u][i]!=fa[v][i]){u=fa[u][i];v=fa[v][i];}
	}
	return fa[u][0];
}

int check(int a,int f,int b)
{
	if(dep[b]<dep[a]&&dep[b]>=dep[f])
	{
		for(int i=0;i<=20;i++)
		{
			if((dep[a]-dep[b])&(1<<i))a=fa[a][i];
		}
		if(a==b)return 1;
		else return 0;
	}
	return 0;
}

int main()
{
	int n=read(),m=read();
	int A,B,C;
	A=read();B=read();C=read();
	for(int i=1;i<=m;i++)
	{
		int a,b;
		a=read();b=read();
		edge[a]+=b;
		edge[b]+=a;
	}
	cnt=n;
	for(int i=1;i<=n;i++)
	{
		if(!dfn[i])num=0,dfs(i);
	}
	dfs2(1,0);
	int LCA=lca(A,C);
	for(int v:dcc[B])
	{
		if(check(A,LCA,v)||check(C,LCA,v)){cout<<"Yes"<<'\n';exit(0);}
	}
	cout<<"No"<<'\n';
	return 0;
}
```



$\mathit{p}$
