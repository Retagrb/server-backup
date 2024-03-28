# 【题解】P2167 [SDOI2009]Bill的挑战

挺好的一道状压 DP。可惜我脑子有病。（）

## 题目链接

[P2167 [SDOI2009]Bill的挑战](https://www.luogu.com.cn/problem/P2167)

[P3225 [HNOI2012]矿场搭建](https://www.luogu.com.cn/problem/P3225)

## 题意概述

给出 $N$ 个长度相同的字符串（由小写英文字母和 `?` 组成），$S_1,S_2,\dots,S_N$，求与这 $N$ 个串中的刚好 $K$ 个串匹配的字符串 $T$ 的个数，答案对 $1000003$ 取模。

若字符串 $S_x(1\le x\le N)$ 和 $T$ 匹配，满足以下条件：
\1. $|S_x|=|T|$。
\2. 对于任意的 $1\le i\le|S_x|$，满足 $S_x[i]= \texttt{?}$ 或者 $S_x[i]=T[i]$。

其中 $T$ 只包含小写英文字母。

## 数据范围

- 对于 $30\%$ 的数据，$N\le5$，$|S_i|\le20$；
- 对于 $70\%$ 的数据，$N\le13$，$|S_i|\le30$；
- 对于 $100\%$ 的数据，$1\le T\le 5$，$1\le N \le15$，$1\le|S_i|\le50$。

## 思路分析

首先看到这个数据范围肯定想到的是状压，并且要状压的是选了哪些字符串。

那么定义 $dp_{sta}$ 表示选择 $sta$ 这个状态的字符串的方案数是多少。

然后就会发现不太好转移，因为我们只能确定 $sta$ 这个状态构成的字符串对应的方案数是多少，而对于不同的 $sta$，可能会有相同的一个字符串 $T$ 满足条件，会算重。

那么我们考虑在状态中加入一维：定义 $dp_{i,sta}$ 表示考虑到了字符串的第 $i$ 位，当前选的状态是 $sta$ 时，方案数是多少。

考虑如何转移。

我们考虑对于一个状态 $dp_{i,sta}$ 它能转移到哪儿去。

首先肯定是要转移到某一个 $dp_{i+1,nxt}$ 上去。那么 $nxt$ 是什么呢。

这时候我们要用到一个辅助数组。

定义 $f_{i,j}$ 表示的是考虑到了字符串的第 $i$ 位，这一位上可以与 $j$ 对应字符匹配的 $S$ 有哪些。

考虑对于一个 $j$，这一位上是什么的时候可以与它匹配，比较显然的是要么这一位是 `?` 要么是 $j$ 对应的字符。

那么可以直接枚举 $i,j$ 并枚举每一个字符串直接判断这一位是否与 $j$ 匹配即可，可以预处理。

有了这个 $f$ 数组，我们也就可以轻松的转移 $dp$ 了。

我们直接考虑下一位填每一个字符 `a` 到 `z` 时对应的方案数是多少即可。
$$
dp[i+1][sta~ \mathrm and~ f[i+1][ch]]+=dp[i][j]
$$
初始状态：$dp_{i,(1<<n)-1}=1$。

那么最后的答案就是对于每一个状态 $sta$ 二进制下 $1$ 的个数恰好为 $k$ 的所有的 $dp_{len,sta}$ 之和。其中 $len$表示每一个字符串的长度。 

## 代码实现

```cpp
//luoguP2167
#include<cstdio>
#include<iostream>
#include<cstring>
#define int long long
using namespace std;
const int mod=1000003;
const int maxn=16;
const int maxm=55;
string s[maxn];
int f[maxm][maxm],dp[maxm][1<<maxn],val[1<<maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

signed main()
{
	int T=read();
	while(T--)
	{
		memset(dp,0,sizeof(dp));
		memset(f,0,sizeof(f));
		memset(val,0,sizeof(val));
		int n,k;
		n=read();k=read();
		for(int i=0;i<n;i++)cin>>s[i],s[i]='%'+s[i];
		int len=s[0].size()-1;
		for(int sta=0;sta<(1<<n);sta++)
		{
			for(int i=0;i<n;i++)
			{
				if(sta&(1<<i))val[sta]++;
			}
		}
		for(int i=1;i<=len;i++)
		{
			for(char ch='a';ch<='z';ch++)
			{
				for(int j=0;j<n;j++)
				{
					if(s[j][i]=='?'||s[j][i]==ch)f[i][ch-'a']|=(1<<j);
				}
			}
		}
		dp[0][(1<<n)-1]=1;
		for(int i=0;i<len;i++)
		{
			for(int sta=0;sta<(1<<n);sta++)
			{
				if(!dp[i][sta])continue;
				for(char ch='a';ch<='z';ch++)
				{
					(dp[i+1][sta&f[i+1][ch-'a']]+=dp[i][sta])%=mod;
				}
			}
		}
		int ans=0;
		for(int sta=0;sta<(1<<n);sta++)
		{
			if(val[sta]==k)(ans+=dp[len][sta])%=mod;
		}
		cout<<ans<<'\n';
	}
	return 0;
}
```

