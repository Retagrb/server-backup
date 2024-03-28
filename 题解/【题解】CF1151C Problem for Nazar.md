# 【题解】CF1151C Problem for Nazar

距离 CSP 剩下 10 天了，据说考前写题解可以增加 RP~所以我来写一篇题解+水点贡献分~

看题解区没有用二分答案来解决这道题的，我来提供一个二分做法。

---

## 题目链接

[CF1151C Problem for Nazar](https://www.luogu.com.cn/problem/CF1151C)

## 题意概述

设正奇数集合为 $\mathrm{A}$，正偶数集合为 $\mathrm{B}$，这两个集合是无限集。

在黑板上写了无数轮数，第 $i$ 轮写下了 $2^{(i-1)}$ 个数.

当$i$为奇数时，从集合 $\mathrm{A}$ 中向后取数，当 $i$ 为偶数时，从集合 $\mathrm{B}$ 中向后取数。

求黑板上第 $l$ 个数到第 $r$ 个数的和，模 $\mathrm{1000000007}$（$10^9+7$）

## 数据范围

$1 \le l,r \le 10^{18}$

## 思路分析

首先看到这个数据范围，可以考虑到时间复杂度大概率是 $\log n$ 这个级别的。

求的是下标为区间 $[l,r]$ 的和，那我们很自然联想到差分：可以先把区间 $[1,r]$ 的和求出来减去区间 $[1,l-1]$ 的和。

那么问题就转化为怎么求出前 $x$ 个数的和。

观察题目条件可以发现，黑板上的数无非就是一串奇数+一串偶数+一串奇数……这样排列的。

且**相邻的两串奇数**之间一定是连续的，也就是假如这一串奇数的结尾是 $k$，那么**与它相邻的下一串奇数**的第一个数一定是 $k+2$。

相邻的两串偶数之间也同理。

> 注意：这里的相邻的两串奇数与相邻的两串数含义不同，每相邻的两串奇数之间隔着一串偶数。

那么我们能否把前 $x$ 个数的和转化为所有的奇数串之和加上所有的偶数串之和呢？

答案是肯定的。

考虑最终形成的序列一定是这样的：

$2^0$ 个奇数+$2^1$ 个偶数+$2^2$ 个奇数+$2^3$ 个偶数+ $\cdots$ +$2^{(i-1)}$ 个不知道是奇数还是偶数的一串数+某个数 $k$ 到 $x$。

很显然第 $i$ 串数的奇偶性取决于 $i$ 的奇偶性，并与 $i$ 的奇偶性相同。

这里的 $k$ 是第 $i-1$ 串数的最后一个数 +2。

那么我们其实可以考虑把前 $i$ 串数的和求出来，然后再单独把从 $k$ 到 $x$ 的和求出来然后相加。

那么我们首先要求出这个 $i$。

因为前 $i$ 个串的长度和为 $2^0+2^1+2^2+\cdots+2^{(i-1)}=2^i-1$，所以我们可以对于每个 $x$ 直接二分求出这个 $i$ 是多少。

具体代码如下：

```cpp
int now=0,ret=0;
for(int step=(1ll<<60);step>=1;step>>=1)
{
	if(now+step<=60&&(1ll<<(now+step))-1<x)now+=step;
}
```

（这里采用的是倍增二分）

最后得到的这个 $now$ 就是我们所说的 $i$。

由于前面已经提到过，相邻的两串奇数/偶数之间都是连续的。

那我们可以直接将前 $i$ 串数的和分成奇数列和偶数列来求。

显然，对于前 $i$ 串数，有 $i/2$ 串偶数列，有 $i-i/2$ 串是奇数列。

那么我们可以预处理出来奇数列和偶数列单独的和。

用 $sum1_i$ 表示整个序列中前 $i$ 串奇数列中有多少个数，$sum2_i$ 表示整个序列中前 $i$ 串偶数列中有多少个数，预处理如下：

```cpp
void init()
{
	int cnt1=0,cnt2=0;
	for(int i=1;i<=60;i++)//之所以是 60，因为 2^60-1 已经大于 1e18 了，所以最多有 60 串数。
	{
		if(i&1)
		{
			cnt1++;
			sum1[cnt1]=sum1[cnt1-1]+(1ll<<(i-1));
		}
		else
		{
			cnt2++;
			sum2[cnt2]=sum2[cnt2-1]+(1ll<<(i-1));
		}
	}
}
```

那么对于前 $i$ 串数，有 $sum2[i/2]$ 个偶数，$sum1[i-i/2]$ 个奇数。

接下来要利用一些等差数列的知识：

- 等差数列第 $n$ 项为 $a_1+nd$（$a_1$ 表示首项，$d$ 表示公差）。

- 等差数列前 $n$ 项的和为 $\frac{(a_1+a_n)*n}{2}$。

那么第 $sum2[i/2]$ 个偶数就为 $sum2[i/2] \times 2$，第 $sum1[i-i/2]$ 个奇数就为 $sum1[i-i/2] \times 2-1$。

所以前 $i$ 串数中偶数列之和为 $s_1=\frac{(2+sum2[i/2]\times 2) \times sum2[i/2]}{2}$，奇数列之和为 $s2=\frac{(1+sum1[i-i/2]*2-1) \times sum1[i-i/2]}{2}$

然后对于最后 $k$ 到 $x$ 的那一段和，可以先把 $k$ 求出来：当 $i$ 为奇数时，$k=sum2[i/2]\times 2+2$，当 $i$ 为偶数时，$k=sum1[i-i/2] \times 2+2$。然后再把 $x$ 求出来：当 $i$ 为奇数时，$k=sum2[i/2] \times 2+(x-2^i+1)\times 2$，当 $i$ 为偶数时，$k=sum1[i-i/2] \times 2+(x-2^i+1)\times 2$。

然后从 $k$ 到 $x$ 也可以直接等差数列求和，设结果为 $s$。

那么最后答案就是 $s1+s2+s$。

## 易错点

别忘了及时取模以及负数保护！！！防止爆 long long！~~我在这挂了 2h……老毛病又犯了。。~~

## 代码实现

```cpp
//CF1151C
#include<cstdio>
#include<iostream>
#define int long long
//#define debug
using namespace std;
const int maxn=105;
const int mod=1e9+7;
int sum1[maxn],sum2[maxn],n;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

void init()
{
	int cnt1=0,cnt2=0;
	for(int i=1;i<=60;i++)
	{
		if(i&1)
		{
			cnt1++;
			sum1[cnt1]=sum1[cnt1-1]+(1ll<<(i-1));
		}
		else
		{
			cnt2++;
			sum2[cnt2]=sum2[cnt2-1]+(1ll<<(i-1));
		}
	}
}

int work(int x)
{
	int now=0,ret=0;
	for(int step=(1ll<<60);step>=1;step>>=1)
	{
		if(now+step<=60&&(1ll<<(now+step))-1<x)now+=step;
	}
	int L=now;
	int re=x-(1ll<<L)+1;
	int tt=L>>1;
	int num=sum2[tt];
	int tot1=num*2;
	int s1=(2+tot1)/2%mod*(num%mod)%mod;
	int kk=L-tt;
	num=sum1[kk];
	int tot2=num*2-1;
	int s2=(1+tot2)/2%mod*(num%mod)%mod;
	if(L&1)
	{
		(ret=(s1+s2)%mod+(tot1+2+tot1+re*2)/2%mod*(re%mod)%mod)%=mod;
	}
	else
	{
		(ret=(s1+s2)%mod+(tot2+2+tot2+re*2)/2%mod*(re%mod)%mod)%=mod;
	}
	return ret;
}

signed main()
{
	init();
	int l,r;
	l=read();r=read();
	cout<<(work(r)-work(l-1)+mod)%mod<<'\n';
	return 0;
}
```

