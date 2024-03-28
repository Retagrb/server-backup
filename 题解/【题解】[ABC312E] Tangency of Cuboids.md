# 【题解】[ABC312E] Tangency of Cuboids

少见的 at 评分 $2000+$ 的 ABC E 题，非常巧妙的一道题。

特别鸣谢：@[dbxxx](https://www.luogu.com.cn/user/120868) 给我讲解了他的完整思路。

## 题目链接

[ABC312E - Tangency of Cuboids](https://atcoder.jp/contests/abc312/tasks/abc312_e)

## 题意概述

给定三维空间中的 $n$ 个长方体，每个长方体以其一条体对角线的两个端点的坐标形式给出，即对于每一个长方体 $i$，给定其体对角线端点的坐标 $(x_{i,1},y_{i,1},z_{i,1})$ 和 $(x_{i,2},y_{i,2},z_{i,2})$。

要求对于给定的每一个长方体，求出给定的其它长方体中，与其共享一个面的长方体数量。

具体地说，对于每个 $i(1 \le i \le n)$，找到 $1≤j≤n$ 且 $j≠i$ 的 $j$ 的数量，使得第 $i$ 个长方体和第 $j$ 个长方体的表面有一个正面积的交集。

**题目保证每个长方体两两不交，即对于任意两个长方体，它们的交集体积为 $0$。**

## 数据范围

- $1\le n \le 10^5$
- $0 \le x_{i,1}<x_{i,2}\le 100$
- $0 \le y_{i,1}<y_{i,2}\le 100$
- $0 \le z_{i,1}<z_{i,2}\le 100$

## 思路分析

$n$ 的范围很大，但是观察长方体体对角线端点的坐标范围只有 $100$，容易想到最终的时间复杂度应该是 $100^3$ 或者 $100^4$，启示我们要处理**单位小正方体**的信息。

> 注：单位小正方体指的是坐标范围内棱长为 $1$ 的正方体。

可以考虑将题目中所有已知的长方体，都分解成多个单位小正方体处理。即，对于长方体 $i$，我们把这个长方体中包含所有单位小正方体都染色成 $i$。

换句话说，我们定义体对角线端点坐标为 $(i,j,k)$ 和 $(i+1,j+1,k+1)$ 的单位小正方体的坐标为 $(i,j,k)$，$col_{i,j,k}$ 表示坐标为 $(i,j,k)$ 的单位小正方体被染成的颜色。那么对于体对角线端点坐标为 $(x_{t,1},y_{t,1},z_{t,1})$ 和 $(x_{t,2},y_{t,2},z_{t,2})$ 的长方体 $t$，就将所有坐标为 $(i,j,k)$，其中 $x_{t,1}\le i <x_{t,2},y_{t,1} \le j < y_{t,2},z_{t,1} \le k < z_{t,2}$ 的单位小正方体都染成 $t$，即让 $col_{i,j,k}=t$。

> 注意：这里 $i,j,k$ 条件是 $x_{t,1}\le i <x_{t,2},y_{t,1}\le j<y_{t,2},z_{t,1} \le k <z_{t,2}$，而不是 $x_{t,1}\le i \le x_{t,2},y_{t,1}\le j \le y_{t,2},z_{t,1} \le k \le z_{t,2}$。因为当 $i=x_{t,2}$ 或 $j=y_{t,2}$ 或 $k=z_{t,2}$ 时，对应的单位小正方体已经不属于这个长方体内了。所以必须是小于，不能是小于等于。

接下来，我们枚举坐标范围（即 $[0,100)$ 内）的每一个单位小正方体的坐标 $(i,j,k)$，那么如果这个小正方体和它相邻小正方形，即 $(i+1,j,k)$ 或 $(i,j+1,k)$ 或 $(i,j,k+1)$ 颜色不同，则说明它们所在的长方体有公共面。那么这样的方法就可以找到所有的有公共面长方体。

我们考虑对于每一个长方体开一个 set，如果长方体 $i$ 与长方体 $j$ 有公共面。那么就往 $i$ 对应的 set 里扔一个 $j$，同时往 $j$ 对应的 set 里面扔一个 $i$，由于 set 无重复元素，这样顺便也完成了去重。那么最后输出 $1$ 到 $n$ 的每个正方体 $i$ 的 set 的 size 就好了。

>注意：如果题目不保证给定长方体两两不交，就不能这么做了。因为如果有交，同一个单位小正方形就可以同时被染成多种颜色，染色复杂度就变成 $O(100^3 n)$，就不能通过这道题了。

时间复杂度 $O(100^3 \log n)$（set 是 $\log n$ 的复杂度）。

## 代码实现

```cpp
//E
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<set>
using namespace std;
const int maxn=1e5+10;
const int maxx=105;
int X1[maxn],X2[maxn],Y1[maxn],Y2[maxn],Z1[maxn],Z2[maxn];
int col[maxx][maxx][maxx];

set<int>s[maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

void work(int xx,int xy,int yx,int yy,int zx,int zy,int t)
{
	for(int i=xx;i<xy;i++)
	{
		for(int j=yx;j<yy;j++)
		{
			for(int k=zx;k<zy;k++)
			{
				col[i][j][k]=t;
			}
		}
	}
}

int main()
{
	int n=read();
	for(int i=1;i<=n;i++)
	{
		X1[i]=read();Y1[i]=read();Z1[i]=read();
		X2[i]=read();Y2[i]=read();Z2[i]=read();
		work(X1[i],X2[i],Y1[i],Y2[i],Z1[i],Z2[i],i);
	}
	for(int i=0;i<100;i++)
	{
		for(int j=0;j<100;j++)
		{
			for(int k=0;k<100;k++)
			{
				if(col[i][j][k]!=col[i+1][j][k])
				{
					if(col[i][j][k]&&col[i+1][j][k])
					{
						s[col[i][j][k]].insert(col[i+1][j][k]);
						s[col[i+1][j][k]].insert(col[i][j][k]);
					}
				}
				if(col[i][j][k]!=col[i][j+1][k])
				{
					if(col[i][j][k]&&col[i][j+1][k])
					{
						s[col[i][j][k]].insert(col[i][j+1][k]);
						s[col[i][j+1][k]].insert(col[i][j][k]);
					}
				}
				if(col[i][j][k]!=col[i][j][k+1])
				{
					if(col[i][j][k]&&col[i][j][k+1])
					{
						s[col[i][j][k]].insert(col[i][j][k+1]);
						s[col[i][j][k+1]].insert(col[i][j][k]);
					}
				}
			}
		}
	}
	for(int i=1;i<=n;i++)cout<<s[i].size()<<'\n';
	return 0;
}
```

