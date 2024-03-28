# 【题解】CF754D Fedor and coupons

## 题目链接

[CF754D Fedor and coupons](https://www.luogu.com.cn/problem/CF754D)

[CF1029C Maximal Intersection](https://www.luogu.com.cn/problem/CF1029C)

后者是前者的加强版。

## 思路分析

最开始，先考虑不删区间 $(k=0)$ 的情况：

也就是给你一大堆区间，让你找他们的交集。

这个还是比较好想的，我们刚开始让第二个区间与第一个区间相交，得到的是一个以 $\max(l_1,l_2)$ 为左端点，以 $\min(r_1,r_2)$ 为右端点的区间，然后再让这个区间与后面的区间相交，每次得到的交集都是以两者 $l$ 取 $\max$，$r$ 取 $\min$ 的区间。

所以我们找位于**最右边**的 $l$，位于**最左边**的 $r$，得到的交集为 $[\max_l,\min_r]$ 那么 $r-l+1$ 即为答案。

如果 $l>r$，那么答案为 $0$。

---

再考虑 $k=1$ 的情况（即 CF1029C）：

也就是删去一个区间，然后使得剩下的区间交集最大。

比较好想的就是直接枚举删去了哪个区间，然后考虑剩下区间的交集。

剩下区间的交集我们每次可以用 $k=0$ 的思路做，但是时间复杂度过高。

我们考虑删掉一个区间与没有删去区间时的原答案有什么变化：

假设原答案的交集区间的左端点是 $L$，右端点是 $R$。

那么如果删掉的这个区间左端点 $l=L$，且所有区间中只有这一个 $l=L$ 的区间，那么删掉这个区间后，新的答案交集左端点就变成了所有区间中第二大的左端点 $ml$。

同理，如果删掉的这个区间右端点 $r=R$，且所有区间中只有这一个 $r=R$ 的区间，那么删掉这个区间后，新的答案交集右端点就变成了所有区间中第二小的右端点 $mr$。

综上所述，我们可以遍历所有的区间，然后处理出所有区间中左端点最大和次大，右端点最小和次小，然后依次考虑删去每个区间后对答案的影响，所有情况下的答案取最大值即可。

具体见下方代码。（这是 $k=1$ 情况下的**第一种思路**）

---

最后考虑 $k>1$ 的情况（即 CF754D）：

也就是删去 $k$ 个区间，然后使得剩下的区间交集最大。

根据上述两种情况，我们很自然想到：删去了 $k$ 个区间，要使得最后剩下的区间交集最大，那么答案区间左端点就是剩下区间左端点最大值 $\max_l$，右端点就是剩下区间右端点最小值 $\min_r$。

$k=1$ 的时候我们是处理了所有区间的最大和次大，那么这个 $k>1$ 怎么办呢，那按照 $k=1$ 的思路，我们应该处理前 $k+1$ 大。

前 $k$ 大问题可以让人联想到优先队列。但是怎么用呢？

如果按照 $k=1$ 的思路继续走下去，我们自然是开一个大根堆，一个小根堆，一个维护 $l$ 的前 $k+1$ 大，一个维护 $r$ 的前 $k+1$ 小。接下来枚举删掉的 $k$ 个区间……这时候就存在问题了，之前因为 $k=1$，所以枚举区间是 $O(n)$ 的复杂度，可以接受。现在 $k$ 本身就是 $3e5$ 这个级别的，直接枚举复杂度显然接受不了，说明直接照搬 $k=1$ 的思路是不可行的。

考虑怎么改进做法。

正难则反，删去 $k$ 个区间，实际上就相当于保留 $m=n-k$ 个区间。

实际上就是找到 $m$ 个区间，使得这 $m$ 个区间 $\min_r-\max_l$ 最小。

那么我们可以将 $l$ 排序，用小根堆来维护 $r$ 的前 $m$ 小。

在排序后的序列中，从 $1$ 到 $n$ 枚举每个区间，并将其右端点丢进小根堆。如果当前堆中元素 $>m$ 就将堆顶元素弹出堆。如果当前堆中元素 $=m$ 就判断当前 $\min_r-\max_l$ 与当前答案 $ans$ 的关系，并更新 $ans$。

那么这个题就做完了。

**注：原题中的 $n$ 实际上相当于这里的 $m$，为了方便照应 $k=0$ 和 $k=1$，我就直接把这种情况当做 $k>1$ 的来处理了。**

---

根据上面的分析过程来看，$k=1$ 的思路几乎与 $k=0$ 一致，只是稍作改变，区别是 $k=1$ 的思路增加了处理最大和次大。同时这个最大和次大又为 $k>1$ 的思路中用优先队列处理前 $m$ 小的 $r$ 提供了启发。

除此之外，$k=1$ 的思路除了我上面提到的这种做法之外，还有另外一种大同小异的处理方法：

枚举删哪个区间，假设我们枚举的区间是 $i$，那么答案区间的左端点就是前 $i-1$ 个区间 $\max_l$ 与第 $i+1$ 到第 $n$ 个区间的 $\max_l$ 取最大值，同理，右端点是前 $i-1$ 个区间 $\min_r$ 与第 $i+1$ 到第 $n$ 个区间的 $\min_r$ 取最小值。

基于此， 我们可以考虑维护前缀/后缀 $\max_l$ 和 $\min_r$，然后枚举删哪个区间，考虑贡献即可。（这是 $k=1$ 情况下**第二种思路**）

同时，由于 $k=1$ 实际上就是 $k>1$ 的特殊情况，依然可以沿用 $k>1$ 的思路，此时就相当于 $m=n-1$。（$k=1$ 情况下**第三种思路**）

## 代码实现

$k=1$ 第一种思路：

```cpp
//CF1029C
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
using namespace std;
const int maxn=3e5+10;
const int INF=1e9;
int l[maxn],r[maxn],ml,mr,mml,mmr;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int main()
{
	int n=read();
	ml=0;mr=INF;
	mml=0;mmr=INF;
	for(int i=1;i<=n;i++)
	{
		l[i]=read(),r[i]=read();
		if(l[i]>=ml)
		{
			mml=ml;
			ml=l[i];
		}
		else if(l[i]>mml)mml=l[i];
		if(r[i]<=mr)
		{
			mmr=mr;
			mr=r[i];
		}
		else if(r[i]<mmr)mmr=r[i];
	}
	int L,R,ans=0;
	for(int i=1;i<=n;i++)
	{
		if(l[i]==ml)L=mml;
		else L=ml;
		if(r[i]==mr)R=mmr;
		else R=mr;
		ans=max(ans,R-L);
	}
	cout<<ans<<'\n';
	return 0;
}
```

$k>1$：

```cpp
//CF754D
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<queue>
#include<algorithm>
#include<vector>
#include<set>
#define mk make_pair
#define pii pair<int,int>
using namespace std;
const int maxn=3e5+10;
int ans;

struct Node{int l,r,id;}a[maxn];

priority_queue<pii>q;

set<int>s;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int cmp(Node a,Node b){return a.l<b.l;}

int main()
{
	int n,k;
	n=read();k=read();
	for(int i=1;i<=n;i++)a[i].l=read(),a[i].r=read(),a[i].id=i;
	sort(a+1,a+n+1,cmp);
	for(int i=1;i<=n;i++)
	{
		q.push(mk(-a[i].r,i));
		if(q.size()>k)q.pop();
		if(q.size()==k&&-q.top().first-a[i].l+1>ans)ans=-q.top().first-a[i].l+1;
	}
	if(ans==0)
	{
		cout<<ans<<'\n';
		for(int i=1;i<=k;i++)cout<<i<<" ";
		exit(0);
	}
	while(!q.empty())q.pop();
	for(int i=1;i<=n;i++)
	{
		q.push(mk(-a[i].r,i));
		s.insert(a[i].id);
		if(q.size()>k)s.erase(s.find(a[q.top().second].id)),q.pop();
		if(q.size()==k&&-q.top().first-a[i].l+1==ans)
		{
			cout<<ans<<'\n';
			for(int v:s)cout<<v<<" ";
			exit(0);
		}
	}
}
```

$k=1$ 第二种思路：

```cpp
//CF1029C
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
using namespace std;
const int maxn=3e5+10;
int l[maxn],r[maxn],prel[maxn],prer[maxn],sufl[maxn],sufr[maxn];

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int main()
{
	int n=read();
	prer[0]=sufr[n+1]=1e9;
	for(int i=1;i<=n;i++)
	{
		l[i]=read();
		r[i]=read();
		prel[i]=max(prel[i-1],l[i]);
		prer[i]=min(prer[i-1],r[i]);
	}
	for(int i=n;i>=1;i--)
	{
		sufl[i]=max(sufl[i+1],l[i]);
		sufr[i]=min(sufr[i+1],r[i]);
	}
	int ans=0;
	for(int i=1;i<=n;i++)
	{
		int L=max(prel[i-1],sufl[i+1]);
		int R=min(prer[i-1],sufr[i+1]);
		ans=max(R-L,ans);
	}
	cout<<ans<<'\n';
	return 0;
}
```

$k=1$ 第三种思路：

```cpp
//CF754D
//The Way to The Terminal Station…
#include<cstdio>
#include<iostream>
#include<queue>
#include<algorithm>
#include<vector>
#include<set>
#define mk make_pair
#define pii pair<int,int>
using namespace std;
const int maxn=3e5+10;
int ans;

struct Node{int l,r,id;}a[maxn];

priority_queue<pii>q;

set<int>s;

inline int read()
{
	int x=0,f=1;char ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
	return x*f;
}

int cmp(Node a,Node b){return a.l<b.l;}

int main()
{
	int n,k;
	n=read();k=n-1;
	for(int i=1;i<=n;i++)a[i].l=read(),a[i].r=read(),a[i].id=i;
	sort(a+1,a+n+1,cmp);
	for(int i=1;i<=n;i++)
	{
		q.push(mk(-a[i].r,i));
		if(q.size()>k)q.pop();
		if(q.size()==k&&-q.top().first-a[i].l>ans)ans=-q.top().first-a[i].l;
//		cout<<-q.top().first<<" "<<q.top().second<<'\n';
	}
	if(ans==0)
	{
		cout<<ans<<'\n';
//		for(int i=1;i<=k;i++)cout<<i<<" ";
		exit(0);
	}
	while(!q.empty())q.pop();
	cout<<ans<<'\n';
//	for(int i=1;i<=n;i++)
//	{
//		q.push(mk(-a[i].r,i));
//		s.insert(a[i].id);
//		if(q.size()>k)s.erase(s.find(a[q.top().second].id)),q.pop();
//		if(q.size()==k&&-q.top().first-a[i].l==ans)
//		{
//			cout<<ans<<'\n';
//			for(int v:s)cout<<v<<" ";
//			exit(0);
//		}
//	}
}

```

