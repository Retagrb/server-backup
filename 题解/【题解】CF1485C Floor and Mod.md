 # 【题解】CF1485C Floor and Mod

emmm……NOIP 考前两周，跟 CSP 考前一样（虽然最后并没有去考），写篇题解增加以下 RP（雾）。

提供一篇思路大体和题解区相同但用了二分写法的题解。

## 题目链接

[CF1485C Floor and Mod](https://www.luogu.com.cn/problem/CF1485C)

## 题意概述

求 $1\le a\le x,1\le b\le y$ 且 $\lfloor\frac{a}{b}\rfloor =a\bmod b$ 的 $(a,b)$ 个数。

## 数据范围

- $1 \le x,y \le 10^9$

## 思路分析

首先我们设 $\left\lfloor \dfrac{a}{b}\right \rfloor=k$，则：$a=bk+k=k(b+1)$。也就是说 $a$ 是 $b+1$ 的倍数。

那么题意转化为：

> 在 $[1,x]$ 里找一个 $a$，在 $[1,y]$ 里找一个 $b$，满足 $a$ 是 $b+1$ 的倍数，问有多少对这样的 $(a,b)$。

那么我们考虑对于 $[1,y]$ 里的每一个 $b$，$[1,x]$ 中有多少个 $a$ 满足题意。其实就相当于问 $[1,x]$ 中有多少个 $b+1$ 的倍数，显然有 $\left\lfloor\dfrac{x}{b+1}\right\rfloor$ 个。

那么总的答案就为 
$$
 \sum \limits_{i=1}^y \left\lfloor\frac{x}{i+1}\right\rfloor=\sum \limits_{i=2}^{y+1}  \left\lfloor\frac{x}{i}\right\rfloor
$$
那么可以直接整除分块解决。

~~结果我写完发现样例都过不了。。~~所以显然是有问题的。

我们发现 $b$ 首先不能为 $1$，因为任何数是 $1$ 的倍数，而任何数除以 $1$ 不可能为 $0$。

所以我们的下界应该从 $3$ 开始。

其次，在 $a=bk+k=k(b+1)$ 中，我们忽略了一个重要条件，$k$ 在这里相当于 $a\bmod b$，是余数，而余数不能大于等于除数，即 $k<b$。

所以对于 $[3,y]$ 的每一个 $b$，其实只有 $\left \lfloor \dfrac{a}{b+1}\right\rfloor<b$ 的 $a$ 才满足题意。

那么这样的 $a$ 在 $[1,x]$ 中有 $\left\lfloor\dfrac{\min(b+1)(b-1),x}{b+1}\right\rfloor$ 个。

那么整个答案就为：
$$
\begin{aligned}\sum \limits_{i=2}^y\left\lfloor\dfrac{\min((i+1)(i-1),x)}  {i+1}\right\rfloor&=\sum \limits_{i=3}^{y+1}\left\lfloor\dfrac{\min(i(i-2),x)}  {i}\right\rfloor\\&=\sum \limits_{i=3}^{lim} (i-2)+ \sum \limits_{i=lim+1}^{y+1}\left\lfloor  \dfrac{x}{i}\right\rfloor\\&=\frac{(1+lim-2)\times(lim-3+1)}{2}+\sum  \limits_{i=lim+1}^{y+1}\left\lfloor\dfrac{x}{i}\right\rfloor\end{aligned}
$$
其中 $lim$ 表示的是使得 $i\times(i-2)\le x$ 的最后一个 $i$。

那么 $lim$ 直接枚举/解不等式/二分都可，我这里采用的是二分。

最后的式子中，第一项显然可以 $O(1)$ 求出，第二项显然可以整除分块，于是整道题成功解决。

时间复杂度：$O(T(\log y+\sqrt y))$。

## 易错点

没啥细节，只是需要开 long long。

## 代码实现

```cpp
//CF1485C
#include<iostream>
#include<cstdio>
#include<cmath>
#define int long long
using namespace std;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int work(int n,int k,int lim)
{
	int ret=0;
	for(int l=lim,r;l<=n;l=r+1)
	{
		if(k/l==0)break;
		r=min(k/(k/l),n);
		ret+=(k/l)*(r-l+1);
	}
	return ret;
}

signed main()
{
	int T;
	T=read();
	while(T--)
	{
		int x,y;
		x=read();y=read();
		int now=0;
		for(int step=(1ll<<30);step>=1;step>>=1)
		{
			if(now+step<=(y+1)&&(now+step)*(now+step-2)<=x)now+=step;
		}
		int lim=now;
		cout<<(lim-2)*(lim-1)/2+work(y+1,x,lim+1)<<'\n';
	}
}
```

