# 【题解】CF1110D Jongmah

代码很短，但是思路我怎么也想不到的神仙 DP。

## 题意概述

你在玩一个叫做 Jongmah  的游戏，你手上有 $n$ 个麻将，每个麻将上有一个在 $1$ 到 $m$ 范围内的整数 $a_i$。

为了赢得游戏，你需要将这些麻将排列成一些三元组，每个三元组中的元素是相同的或者连续的。如 $7,7,7$ 和 $12,13,14$ 都是合法的。你只能使用手中的麻将，并且每个麻将只能使用一次。

请求出你最多可以形成多少个三元组。

## 数据范围

- $1 \le n,m \le 10^6$
- $1 \le a_i \le m$

## 思路分析

首先我们定义第 $i$ 个顺子表示三个麻将上写着 $\{i,i+1,i+2\}$ 的三元组，$i$ 的三连击表示三个麻将上都写着 $i$ 的三元组，$cnt_i$ 表示写着 $i$ 的麻将数。

那么有一个结论是：对于每个 $i(1 \le i \le m)$，第 $i$ 个顺子最多只需要两次。因为 $3$ 个第 $i$ 个顺子可以转化为 $3$ 个三连击（$i,i+1,i+2$ 的三连击），所以一定存在最优解满足第 $i$ 个顺子取不超过两次。这是麻将题的常见技巧。

那么我们就可以考虑枚举第 $i$ 个顺子选了多少次（$0,1$ 或 $2$ 次）。

如果直接枚举每个顺子选了多少次，是 $3^m$ 的复杂度，不能接受。考虑通过 DP 优化。

但是如果直接 DP，发现很难进行状态设计，那么我们可以考虑简化问题。

考虑先解决如下问题：

> 如果我们并不是可以选择每个顺子，而是只能选择第 $i$ 个顺子（其中 $i\bmod 3=1$），那么如何确定有多少种合法的选择方案。

这个问题的简化之处就在于，第 $i$ 个的顺子的选择不受任何其它顺子的限制。

那么我们可以直接定义 $dp_i$ 表示选到了第 $i$ 个顺子总共有多少种三元组的选择情况。然后枚举选择次数 $k$，则对于每个 $i$，选择 $i$ 的三连击的个数是 $\left \lfloor\dfrac{cnt_i-k}{3}\right\rfloor$，那么 $dp_i$ 就可以用 $dp_{i-3}$ 加上第 $i$ 个顺子的选择情况再加上 $i,i-1,i-2$ 的三连击的选择情况而得到，那么有：
$$
dp_{i}=\max\limits_{0 \le k \le 2}\{dp_{i-3}+k+\left \lfloor\dfrac{cnt_i-k}{3}\right\rfloor+\left \lfloor\dfrac{cnt_{i-1}-k}{3}\right\rfloor+\left \lfloor\dfrac{cnt_{i-2}-k}{3}\right\rfloor\}
$$
这是只能选择第 $i(i\bmod 3=1)$ 个顺子的情况。

---

那么对于所有 $1\le i \le m$ 的每一个 $i$ 都能选的情况呢？

可以发现这个时候第 $i$ 个的顺子的选择情况和 $i$ 的三连击的选择情况，都受到第 $i-1$ 个顺子的选择情况和第 $i-2$ 的顺子选择情况的影响，这个时候我们就不能直接定义 $dp_i$ 表示选到了第 $i$ 个顺子的方案来解决问题了。我们考虑将第 $i-1$ 个顺子的选择情况和第 $i-2$ 个顺子的选择情况加进状态，由于这里第 $i$ 个顺子的选择情况也会影响到后面第 $i+1$ 个顺子的选择情况，所以也要把它加进状态，即 $dp_{i,j,k,t}$ 表示考虑到了第 $i$ 个顺子，第 $i$ 个顺子选择了 $j$ 次，第 $i-1$ 个顺子选择了 $k$ 次，第 $i-2$ 个顺子选了 $t$ 次的方案数，那么这时候 $dp_{i,j,k,t}$ 就可以直接由 $dp_{i-1,k,t,l}$ 其中 $l$ 是第 $i-3$ 个顺子的选择情况。

这时候我们发现，第 $i-3$ 个顺子的选择情况，对第 $i$ 个顺子的选择情况和 $i$ 的三连击的选择情况没有影响。同时在 $dp_{i-1,k,t,l}$ 的状态里已经包含了第 $i-2$ 个顺子的选择情况，所以我们可以将状态的最后一维删去。那么我们最终确定的 dp 状态就是：$dp_{i,j,k}$ 表示考虑到了第 $i$ 个顺子，第 $i$ 个顺子选择了 $j$ 次，第 $i-1$ 个顺子选择了 $k$ 次的方案数是多少。

那么对于 $dp_{i,j,k}$，就可以由 $dp_{i-1,k,t}$ 转移而来，其中 $t$ 表示第 $i-2$ 个顺子选择了 $t$ 次。然后我们枚举 $j,k,t$，那么 $dp_{i,j,k}$ 就等于 $dp_{i-1,k,t}$ 加上第 $i$ 个顺子选择次数 $j$ 再加上 $i$ 的三连击的选择次数。

---

现在问题就转化为，如何在已知 $j,k,t$ 的情况下，求 $i$ 的三连击可以选择多少次。

假设 $j=1,k=1,t=1$，如图所示：

![img](https://cdn.luogu.com.cn/upload/image_hosting/a9aiqf3g.png)

由该图可知，红色的部分即为三连击的个数，即 $\left\lfloor\dfrac{cnt_i-j-k-t}{3}\right\rfloor$。

那么总的转移方程式就为：
$$
dp_{i,j,k}=\max\limits_{0 \le j,k,t \le 2}\{dp_{i-1,k,t}+j+\left\lfloor\dfrac{cnt_i-j-k-t}{3}\right\rfloor\}
$$
时间复杂度 $O(27n)$。

## 代码实现

```cpp
//CF1110D
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn=1e6+10;
int dp[maxn][3][3],a[maxn],cnt[maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int main()
{
	int n,m;
	n=read();m=read();
	for(int i=1;i<=n;i++)
	{
		a[i]=read();
		cnt[a[i]]++;
	}
	for(int i=1;i<=m;i++)
	{
		for(int j=0;j<3;j++)
		{
			for(int k=0;k<3;k++)
			{
				for(int t=0;t<3;t++)
				{
					if(cnt[i]<j+k+t)continue;
					dp[i][j][k]=max(dp[i][j][k],dp[i-1][k][t]+(cnt[i]-j-k-t)/3+j);
				}
			}
		}
	}
	cout<<dp[m][0][0]<<"\n";
	return 0;
}
```

