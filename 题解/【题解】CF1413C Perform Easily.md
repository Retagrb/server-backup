# 【题解】CF1413C Perform Easily 

写篇题解水水经验~顺便增加一下 RP~

比较套路和简单的一道绿题。

## 题目链接

[Perform Easily - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/CF1413C)

## 题意概述

给你一个长度为 $6$ 的 $a$ 数组，和一个长度为 $n$ 的 $b$ 数组，要求将 $b$ 数组内的每一个数，减去 $a_1\sim a_6$ 中的一个，然后让处理后的数组极差（即最大值与最小值的差）最小。

## 数据范围

- $1\le n \le 1\times 10^6$
- $1 \le a_i,b_i \le 1\times 10^9$

## 思路分析

首先考虑处理后数组的最大和最小值，一定是对于所有的 $i(1 \le i \le n)$，$b_i$ 减去 $a_1 \sim a_6$ 之后所有数中的一个。

那么我们把对于每个 $i$，$b_i$ 减去 $a_1\sim a_6$ 的所有值扔到一个 `vector` 里。为了确定 `vector` 中的每一个数是哪个 $b_i$ 处理得来的，我们给 `vector` 中的每个元素都规定一个 $id$，如果这个数是由 $b_i$ 减去 $a_1$ 到 $a_6$ 中的某一个得来的，我们就让这个数的 $id=i$。

那么问题实际上就转化为：

> 有一个长度为 $6n$ 的数组，且每个元素都有一个 $id$，现在让你从这个数组里面选择一些数，要求：
>
> - 对于所有的 $i(1\le i \le n)$，要求选出来的数中至少有一个数的 $id=i$。
> - 选出来的数在所有合法的选择方案中，极差最小。

定义这个 `vector` 为 $p$。

因为要求的是极差最小，那实际上就是让我们确定选出来数的最大值和最小值。那么想到枚举最小值，那么我们只需要考虑当 $p_i$ 作为最小值时，谁能够成为最小的最大值。

由于题目要求的是极差，我们联想到可以将这个 `vector` 中的元素排序。

所以当 $p_i$ 作为最小值时，选的数一定是在 $p_i$ 右边，如果 $p_j(j>i)$ 作为最大值，那么下标为区间 $[i,j]$ 中的数都可以被选进去，但是我们并不关心选哪些数，只要确定最大值和最小值。也就是说如果区间 $(i,j)$ 中有多个数 $id$ 相同，那么这些数中任选一个都可以。所以对于所有 $1 \le k \le n$，只要区间 $[i,j]$ 中至少存在一个数的 $id=i$ 即可。

也就是说当 $p_i$ 为最小值时，$p_j$ 可以作为最大值当且仅当下标为区间 $[i,j]$ 中数的 $id$ 覆盖了 $1\sim n$。由此可知可以作为最大值的下标 $j$ 一定是从最后一个元素到前面某个点的连续一段区间（因为如果 $[i,k]$ 中数的 $id$ 覆盖了 $1\sim n$，由于 $[i,k+1]$ 包含了区间 $[i,k]$，所以 $[i,k+1]$ 中的数的 $id$ 也一定覆盖了 $1\sim n$），假设这个区间是 $[r,n]$，最小的最大值就是 $p_r$，那么我们从 $l$ 开始往后枚举，顺便记录一下当前的 $id$ 覆盖了 $1\sim n$ 中的多少个数，遇到第一个完全覆盖 $1\sim n$ 的下标就是 $r$。

到目前为止我们已经有了一个做法了：枚举 $p_i$ 作为选出来的数的最小值，然后从 $i$ 开始向后枚举最大值，顺便记录当前区间的 $id$ 覆盖了 $1\sim n$ 中得多少个数，然后遇到第一个完全覆盖的下标 $j$，则 $p_j$ 即为最小的最大值，此时的极差为 $p_j-pi$，那么最后最小的极差就是每个 $p_i$ 作为最小值时最小极差取 $\min$。

但是这样做复杂度是 $O(n^2)$ 级别的，无法接受，考虑优化。

我们发现枚举 $p_i$ 作为最小值，当 $i$ 逐渐增大时，对应最小的最大值下标 $j$ 也一定是单调不降的。这一点很好理解。因为当你 $i$ 变成 $i+1$ 时，也就是 $[i,j]$ 变成 $[i+1,j]$。假设 $p_i$ 的 $id$ 是 $t$，那么如果原先 $[i,j]$ 中 $id$ 为 $t$ 的元素个数大于 $1$，那么满足条件的 $j$ 不变；如果 $id$ 为 $t$ 的个数本来为 $1$，那么 $i+1$ 之后 $id$ 为 $t$ 的个数现在变为 $0$，所以新的 $j>$ 原先的 $j$。

有了单调不降这个特征，我们就可以愉快的使用双指针了，直接用双指针维护当前最小值和最大值的位置 $l,r$。顺便在指针移动的过程中维护当前区间覆盖的 $id$ 的个数即可。

时间复杂度 $O(6n)$。

## 代码实现

```cpp
//CF1413C
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<vector>
#include<algorithm>
#define pii pair<int,int>
#define mk make_pair
#define int long long
using namespace std;
const int maxn=1e6+10;
int b[maxn],a[10],cntnow[maxn];

vector<pii>p;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int cmp(pii a,pii b){return a.first<b.first;}

signed main()
{
	for(int i=1;i<=6;i++)a[i]=read();
	int n=read();
	for(int i=1;i<=n;i++)
	{
		b[i]=read();
		for(int j=1;j<=6;j++)p.push_back(mk(b[i]-a[j],i));
	}
	sort(p.begin(),p.end(),cmp);
	int r=-1;
	int ans=1e18;
	int now=0;
	for(int l=0;l<p.size();l++)
	{
		while((r<(signed)p.size()-1)&&now<n)
		{
			r++;
			cntnow[p[r].second]++;
			if(cntnow[p[r].second]==1)now++;
		}
		if(now>=n)ans=min(ans,p[r].first-p[l].first);
		cntnow[p[l].second]--;
		if(cntnow[p[l].second]==0)now--;
	}
	cout<<ans<<'\n';
	return 0;
}
```

