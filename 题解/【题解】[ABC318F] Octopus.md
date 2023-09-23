# 【题解】[ABC318F] Octopus

## 题目链接

[F - Octopus](https://atcoder.jp/contests/abc318/tasks/abc318_f)

## 题意概述

有个机器人，它有 $n$ 个手臂，第 $i$ 个手臂长度为 $l_i$。同时有 $n$ 个宝藏，第 $i$ 个宝藏的坐标是 $x_i$。

当机器人位于 $k$ 时，它的第 $i$ 条手臂可以够到 $[k-l_i,k+l_i]$ 范围内的宝藏。

机器人的每条手臂只能选择一个宝藏。请问总共有多少个整数坐标，能够让机器人在这个坐标能够拿到所有宝藏？

## 数据范围

- $1 \le n \le 200$
- $- 10^{18} \le x_1 < x_2 < \cdots < x_n \le 10^{18}$
- $1 \le l_1 <l_2 <\cdots <l_n \le 10^{18}$

## 题目分析

注意到 $n$ 的范围只有 $200$，可以想到应该是 $n^3$ 级别的复杂度。

---

由于机器人的每条手臂只能选择一个宝藏，我们考虑对于每一个宝藏 $i(1\le i \le n)$，预处理出来当它被第 $j(1 \le j \le n)$ 条手臂抓取时，$k$ 可能的坐标范围 $ans_{i,j}$，即 $ans_{i,j}=[x_i-l_j,x_i+l_j]$。

那么预处理出来所有宝藏 $i$ 对于每一条手臂 $j$ 抓取时，$k$ 可能的坐标范围就是 $O(n^2)$。

假如我们已经钦定了哪个宝藏被哪个手臂抓取，那么要抓取到所有的宝藏合法的区间范围，就是对于所有的宝藏 $i$ 和要抓取它的手臂 $j$ 对应的 $k$ 的可能的坐标范围的交集，即假设抓取宝藏 $i$ 的手臂 $j=id_i$，那么答案就是 $res=ans_{1,id_1} \cap ans_{2,id_2} \cap \cdots ans_{n,id_n}$。

那么总答案就是对于所有宝藏与手臂配对的情况的 $k$ 的合法范围 $res$ 取一个并集。

由此可以知道，最终的答案区间是若干个区间的并集，并且这些区间的端点一定是某个 $x_i-l_j$ 或 $x_i+l_j$。

> 注意：这里的并集是指答案区间的若干个极大区间的并。

---

【思路一】

可以发现这若干个区间的左端点一定是某个 $x_i-l_j$，右端点一定是 $x_i+l_j$。

那么我们可以分别枚举每个 $x_i-l_j$ 和 $x_i+l_j$，看看他们是否能成为某个区间的左端点/右端点。

如果一个 $L=x_i-l_j$ 满足当 $L$ 作为 $k$ 时可以抓取到所有的 $n$ 个宝藏，而 $L-1$ 作为 $k$ 时不能抓取到所有的 $n$ 个宝藏，那么 $L=x_i-l_j$ 就是最终答案区间所构成的若干个区间并集中其中一个区间的左端点。反之不是。

同理，如果一个 $R=x_i-l_j$ 满足当 $R$ 作为 $k$ 时可以抓取到所有的 $n$ 个宝藏，而 $R+1$ 作为 $k$ 时不能抓取到所有的 $n$ 个宝藏，那么 $R=x_i+l_j$ 就是最终答案区间所构成的若干个区间并集中其中一个区间的右端点。反之不是。

那么我们只需要对于所有的左端点 $L_i$ 和右端点 $R_i$ 排序，然后那么最后答案区间的并集中从左向右的第 $i$ 个区间的大小就为 $R_i-L_i+1$。

还有一个问题是当 $k=k_0$ 时，如何确定是否可以抓取所有的 $n$ 个宝藏。

比较显然的是可以直接贪心从小到大排序每个宝藏到 $k$ 的距离 $|x_i-k|$ 和每个手臂最大能够到的距离 $l_i$，然后让其一一配对。

那么我们就得到了一个 $O(n^3 \log n)$ 的做法。

---

【思路二】

如果我们已经固定了 $k$，那么就可以直接贪心从小到大排序每个宝藏到 $k$ 的距离 $|x_i-k|$ 和每个手臂最大能够到的距离 $l_i$，然后让其一一配对。

由于 $x_i$ 的范围太大，不可能直接从 $-10^{18}$ 枚举到 $10^{18}$，那么考虑优化。

由于最终答案区间是若干个区间的并，所以区间端点只有 $n^2$ 级别。那么我们考虑怎么样的点 $k_0$ 可以作为状态的分界点。

> 注：
>
> 这里状态的定义是指当机器人位于该点时，是否能抓取所有宝藏。
>
> 状态的分界点跟区间的端点不同。状态的分界点是指，当机器的位置从 $k$ 变到 $k+1$，$k$ 是否能抓取宝藏与之前不变，但 $k+1$ 与 $k$ 的状态不同。

即有两种情况。$k=k_0$ 可以抓取到所有的 $n$ 个宝藏，$k=k_0+1$ 不能抓取所有宝藏；或 $k=k_0$ 不能抓取所有宝藏，$k=k_0+1$ 能抓取所有宝藏。

考虑这两种情况：

1. $k=k_0$ 满足条件 $k=k_0+1$ 不满足条件。

   那么一定存在宝藏 $i$ 和手臂 $j$ 使得 $x_i-l_j \le k_0 \le x_i+l_j$ 且 $k_0+1 > x_i+l_j$，解得 $k_0=x_i+l_j$。

2. $k=k_0$ 不满足条件 $k=k_0+1$ 满足条件。

   那么一定存在宝藏 $i$ 和手臂 $j$ 使得 $x_i - l_j \le k_0+1 \le x_i+l_j$ 且 $k_0<x_i-l_j$。解得 $k_0=x_i-l_j-1$。

所以当且仅当 $k=x_i+l_j$ 和 $x_i-l_j-1$ 才有可能是分界点。

那么我们将所有可能的端点排序后放入集合 $S$ 中，相邻两个分界点内的点的状态一定相同。

枚举每一个可能的分界点 $k_0$，若 $k=k_0$ 时能抓取所有宝藏，那么它要么是分界点，即第一种情况，要么不是分界点（$k=k_0+1$ 满足条件），但不管怎么样从这个点 $S_i$ 到上一个可能分界点 $S_{i-1}$ 的这段区域一定都能抓取所有宝藏。那么我们将这段区域的点累加入答案中即可。

时间复杂度同样是 $O(n^3 \log n)$。

## 代码实现

**思路一**

```cpp
//F 解法二
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<vector>
#include<algorithm>
#include<cmath>
#define int long long
using namespace std;
const int maxn=205;
int l[maxn],x[maxn],t[maxn];
int n,ans;

vector<int>ls,rs;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int check(int k)
{
	for(int i=1;i<=n;i++)t[i]=abs(k-x[i]);
	sort(t+1,t+n+1);
	for(int i=1;i<=n;i++)
	{
		if(t[i]<=l[i])continue;
		return 0;
	}
	return 1;
}

signed main()
{
	n=read();
	for(int i=1;i<=n;i++)x[i]=read();
	for(int i=1;i<=n;i++)l[i]=read();
	for(int i=1;i<=n;i++)
	{
		for(int j=1;j<=n;j++)
		{
			int L=x[i]-l[j],R=x[i]+l[j];
			if(check(L)&&(!check(L-1)))ls.push_back(L);
			if(check(R)&&(!check(R+1)))rs.push_back(R);
		}
	}
	sort(ls.begin(),ls.end());
	ls.erase(unique(ls.begin(),ls.end()),ls.end());
	sort(rs.begin(),rs.end());
	rs.erase(unique(rs.begin(),rs.end()),rs.end());
	for(int i=0;i<ls.size();i++)ans+=rs[i]-ls[i]+1;
	cout<<ans<<'\n';
}
```

**思路二**

```cpp
//F
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<vector>
#include<algorithm>
#include<cmath>
#define int long long
using namespace std;
const int maxn=205;
int l[maxn],x[maxn],t[maxn];
int n;

vector<int>s;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int check(int k)
{
	for(int i=1;i<=n;i++)t[i]=abs(k-x[i]);
	sort(t+1,t+n+1);
	for(int i=1;i<=n;i++)
	{
		if(t[i]<=l[i])continue;
		return 0;
	}
	return 1;
}

signed main()
{
	n=read();
	for(int i=1;i<=n;i++)x[i]=read();
	for(int i=1;i<=n;i++)l[i]=read();
	for(int i=1;i<=n;i++)
	{
		for(int j=1;j<=n;j++)
		{
			s.push_back(x[i]+l[j]);
			s.push_back(x[i]-l[j]-1);
		}
	}
	sort(s.begin(),s.end());
	int ans=0;
	for(int i=1;i<s.size();i++)
	{
		if(check(s[i]))ans+=s[i]-s[i-1];
	}
	cout<<ans<<'\n';
	return 0;
}
```

