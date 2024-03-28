# 【题解】[ABC304F] Shift Table

## 题目链接

## 题意概述

Takahashi 和 Aoki 将在接下来的 $N$ 天里兼职工作。 

Takahashi 这 $n$ 天的出勤情况由字符串 $S$ 表示，其中 $S$ 的第 $i$ 个字符是 `#` 则表示他在第 $i$ 天工作，第 $i$ 个字符是 `.` 表示他在第 $i$ 天休息。

Aoki 的出勤情况如下：

- 首先选择一个 $N$ 的正因数 $M$，其中 $M\ne N$；
- 接下来安排 $1$ 到 $M$ 天的出勤情况；
- 最后安排 $M+1$ 到 $N$ 天的出勤情况，使得对于任意 $i(1\le i \le M)$，第 $kM+i$ 天的出勤情况与第 $i$ 天一致。

**注意：不同的 $M$ 可能会形成相同的出勤情况安排。**

求 Aoki 可能的出勤情况安排数量，使得 Takahashi 和 Aoki 至少在每一天都有一人工作。

答案对 $998244353$ 取模。

## 数据范围

- $1\le N \le 10^5$

## 题目分析

首先我们发现，$10^5$ 范围内的数的因子数不超过 $128$ 个。

那么很容易想到去枚举因子 ，考虑 $N$ 的每个因子的贡献。

对于每个因子 $M$，我们可以将 $N$ 天平均分为 $\dfrac{N}{M}$ 份，根据题目要求， 每一份的出勤安排相同。

由于必须保证两人至少在每一天都有一人工作，所以若第 $i$ 天 Takahashi 休息，则 Aoki 在这一天必须出勤。同时，如果确定了第 $i$ 天 Aoki 出勤，那么第 $kM+i$ 天 Aoki 也必须出勤。换句话说，若第 $i$ 天出勤，那么所有使得 $j\bmod M=i \bmod M$ 的第 $j$ 天也必须出勤。

基于此，我们可以定义一个 $vis$ 数组，其中 $vis_i$ 表示每一份的第 $i$ 天是否必须出勤。枚举从 $1$ 到 $N$ 的每一天，若 $S_i$ 为 `.`，则将 $vis_{i\bmod M}$ 设为 $1$。

那么对于 $vis_i=0$ 的 $i$，既可以出勤又可以休息，也就是说第 $i$ 天有两种情况。对于 $vis_i=1$ 的 $i$，必须出勤，只有一种情况。

所以对于每个因子 $M$，如果 $vis_i=0$ 的 $i$ 的个数为 $t$，符合题意的方案数就是 $2^t$。

观察样例可以发现，对于所有的 $M$，答案并非是简单的所有 $M$ 的方案数相加，因为不同的 $M$ 可能会形成相同的出勤情况安排。

我们考虑什么样的 $M$ 可以形成相同的出勤安排。容易发现，对于因子 $M_i$ 和 $M_j$，若 $M_i$ 是 $M_j$ 的倍数，则 $M_i$ 的方案数完全包含 $M_j$ 的方案数。那么为了避免重复计数，我们应该将 $M_i$ 的方案数减去 $M_j$ 的方案数计入总答案。

所以我们可以对于每一个因子 $M$ 的方案数，减去所有 $N$ 的因子中同时也是 $M$ 的因子的方案数，即为 $M$ 单独对答案的贡献。

最后，所有的因子贡献之和即为答案。

时间复杂度 $O(128N)$。

## 代码实现

```cpp
//F
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<vector>
#include<set>
#define int long long
using namespace std;
const int maxn=1e5+10;
const int mod=998244353;
set<int>p,q;
int vis[maxn],sum[maxn];//vis[i] 表示的是每一份的第 i 天是否必须出勤，sum[i] 表示的是因子 i 的贡献。

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

signed main()
{
	int n=read();
	string s;
	cin>>s;
	s='%'+s;
	//预处理出 n 的所有因子
	for(int i=1;i<n;i++)
	{
		if(n%i==0)p.insert(i);
	}
	//枚举所有的因子，考虑每个因子的贡献。
	int ans=0;
	for(int v:p)
	{
		for(int i=1;i<=v;i++)vis[i]=0;
		for(int i=1;i<=n;i++)
		{
			int t=i%v;
			if(t==0)t=v;
			if(s[i]=='.')vis[t]=1;
		}
		sum[v]=1;
		for(int i=1;i<=v;i++)
		{
			if(!vis[i])(sum[v]*=2)%=mod;
		}
        //容斥，计算每个因子单独的贡献。
		for(int vv:p)
		{
			if(vv==v)break;
			if(v%vv==0)sum[v]=(sum[v]-sum[vv]+mod)%mod;
		}
		(ans+=sum[v])%=mod;
	}
	cout<<ans<<'\n';
	return 0;
}
```









