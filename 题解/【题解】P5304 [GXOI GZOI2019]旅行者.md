# 【题解】P5304 [GXOI/GZOI2019]旅行者

一道利用 dijkstra 的很妙的图论题！

加深了我对于 dijkstra 的理解。

（于是在做完这道题两天后的模拟赛中遇到了和它套路几乎一样的，我却甚至没有想到用最短路……）

所以写个题解记录一下吧。

---

## 题目链接

[[GXOI/GZOI2019]旅行者 - 洛谷](https://www.luogu.com.cn/problem/P5304)

## 题意概述

给定一张 $n$ 个点 $m$ 条边的带权无向图，其中有 $k$ 个关键点，求这 $k$ 个关键点两两之间最短路的最小值。

## 思路分析

### 解法一

两两之间最短路，单纯的暴力做法是考虑直接以每个点为起点跑一遍 dijkstra，但这样的做法显然太慢。

暴力做法的瓶颈在于每次 dijkstra 只能算出一个点到其它点的最短路，而不能算出多个点到其它点的最短路。

那么给我们启发——我们是否能在一次 dijkstra 中求出多个点到其它点的最短路呢？

联想到我们求解 dijkstra 的过程，第一步是将起点入队，并将起点的 $dis$ 设为 $0$。

那么要将单源转化为多源，就可以将要作为起点的多个点都入队，然后将这些点的 $dis$ 都设为 $0$。

为了方便起见，实际上我们可以直接建立一个虚点 $0$，然后将 $0$ 与所有要作为起点的点都连一条边权为 $0$ 的有向边，那么实际上又可以重新把多源转化为单源了。

我们现在可以用 dijkstra 求多源最短路了。现在问题就是，本题如何转化可以使得在复杂度范围内，进行多次 dijkstra，求得所有的 $k$ 个点两两之间的最短路。

提供一种做法：

我们可以对于每一个点，按照二进制下的每一位，将它分为两个集合，这一位为 $0$ 的放在 $S$ 中，这一位为 $1$ 的放在 $T$ 中，然后我们只需要每次求两个集合间的最短路即可。

由于两个不同的数二进制下一定至少有一位不同，所以当把二进制下的每一位都进行分组之后，对于任意两个点，至少会有一次分在不同的集合，也就是说在统计答案时它们两点之间的最短路一定会被算进去。

所以我们只需要做 $\log n$ 次 dijkstra，然后每次都采用上面的做法：建立两个新的虚点将其编号为 $0$ 和 $n+1$，将虚点 $0$ 向 $S$ 中的所有点连一条边权为 $0$ 的边，将 $T$ 中的所有点都向虚点 $n+1$ 连一条边权为 $0$ 的边，然后以 $0$ 为起点做 dijkstra，每次做完后都用当前的 $dis_{n+1}$ 更新 $ans$ 即可。

需要注意的是这是一张有向图，所以还需要反向建图然后跑一遍 dijkstra 再更新答案。

（如果是无向图的话由于边没有区别所以不需要再反向跑一遍 dijkstra）

时间复杂度：$O(T n\log n\log k)$。5s 时限完全可过。

### 解法二

事实上我们可以只跑两遍 dijkstra。

第一次 dijkstra 将所有的兴趣城市都初始加入队列中，也就是相当于以所有的兴趣城市为起点，处理出来它们到其它点的最短路，维护数组 $dis1$ 表示从兴趣城市出发到每个点的最短路，并维护一个 $col1_i$ 数组表示到 $i$ 的最短路对应的起点是什么（从哪来的）。

同理，第二次 dijkstra 建反图，处理出来 $dis2$ 表示从每个点出发到兴趣城市的最短路，并维护处一个 $col2_i$ 数组表示从 $i$ 出发的最短路对应的终点是什么（到哪去）。

然后我们遍历每条边，对于一条边 $(u,v)$，若 $col1_u \ne col2_v$（非环）则用 $dis1_u+dis2_v+w(u,v)$ 更新 $ans$ 即可。

$ans$ 就是所有可能答案的最小值。

时间复杂度 $O(T n\log n)$。

显然看上去第二种解法复杂度确实更优秀，但第一种解法确实是提供了一种很妙的划分集合的方法，二者都是为了将暴力 dijkstra 次数减少，从而达到减小时间复杂度的目的。

## 易错点

- 对于解法一，我们在每次进行虚点连边之后都要这些边删去消除对后续的影响，不要遗忘。

- 对于解法二，注意区分出来两个染色数组 $col1$ 和 $col2$ 的实际含义：即 $co11$ 表示从哪来，$col2$ 表示到哪去。

## 拓展

这道题实际上将 dijkstra 原本的特性——单源最短路拓展到了多源最短路径上。

这意味着我们可以处理一些以前的传统 dijkstra 处理不了的多源问题。

例如：无向图最小环问题。

无向图最小环的传统解法是 Floyd 算法。

思想：对于一个无向图上的简单环 $i \rightarrow j \rightarrow k \rightarrow i$ 来说，环的长度实际上就为 $w(i,j)+dis(j,k)+w(k,i)$。

$w(i,j)$ 和 $w(k,i)$ 是固定的，在求解 $dis(j,k)$ 时我们采用的是 Floyd 算法。

由于最后是最小环，所以我们取得是最后答案的最小值。

那么现在经过优化了 dijkstra 之后，还有一种无向图求最小环的使用 dijkstra 的新做法供参考。

我们只需要将此题的求解过程中稍作变形即可解决求无向图最小环的问题。

发现相对于本题，求无向图最小环需要多加上两条边的边权，那么我们可以巧妙地将上述解法一求解时虚点 $0$ 连向所有关键城市的边权改为 $w(i,j)$，将所有从关键城市连向虚点 $n+1$ 的边边权改为 $w(j,k)$ 即可，最后答案依然是所有 $dis_{n+1}$ 中的最小值。

需要注意的另一个差异是，本题是有向图，而我们现在要解决的是无向图最小环，由于无向图任意一条边正反都是等价的，所以我们只需要跑一次 dijkstra 就行，不需要跑两次。

而对于解法二，我们可以在每次 dijkstra 将所有兴趣城市入队时，直接将它们的 $dis$ 设置为 $1$ 到这些城市的边权即可。

## 代码实现

### 解法一

```cpp
//luoguP5304
#include<iostream>
#include<cstdio>
#include<cstring>
#include<queue>
#define int long long
using namespace std;
const int maxn=1e5+10;
const int INF=2e18;
int a[maxn],cnt[maxn],dis[maxn],vis[maxn];
int ans=INF;
int n,m,k;

struct Node{
    int v,w;
    bool operator<(const Node &t)const
    {
        return w>t.w;
    }
};

basic_string<Node>edge[maxn];

inline int read()
{
    int x=0,f=1;char ch=getchar();
    while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
    while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
    return x*f;
}

void dijkstra()
{
    priority_queue<Node>q; 
    memset(vis,0,sizeof(vis));
    for(int i=0;i<=n+1;i++)dis[i]=(1ll<<50)-1;
    dis[0]=0;
    q.push(Node{0ll,dis[0]});
    while(!q.empty())
    {
        Node now=q.top();
        q.pop();
        if(vis[now.v])continue;
        vis[now.v]++;
        for(Node y:edge[now.v])
        {
            if(dis[y.v]>dis[now.v]+y.w)
            {
                dis[y.v]=dis[now.v]+y.w;
                q.push(Node{y.v,dis[y.v]});
            }
        }
    }
}

void dijkstra2()
{
    priority_queue<Node>q;
    memset(vis,0,sizeof(vis));
    for(int i=0;i<=n+1;i++)dis[i]=(1ll<<50)-1;
    dis[n+1]=0;
    q.push(Node{n+1,dis[n+1]});
    while(!q.empty())
    {
        Node now=q.top();
        q.pop();
        if(vis[now.v])continue;
        vis[now.v]++;
        for(Node y:edge[now.v])
        {
            if(dis[y.v]>dis[now.v]+y.w)
            {
                dis[y.v]=dis[now.v]+y.w;
                q.push(Node{y.v,dis[y.v]});
            }
        }
    }
}

signed main()
{
    int T;
    T=read();
    while(T--)
    {
        n=read();m=read();k=read();
        ans=INF;
        for(int i=1;i<=m;i++)
        {
            int u,v,w;
            u=read();v=read();w=read();
            edge[u]+=Node{v,w};
//            edge[v]+=Node{u,w};
        }
        for(int i=1;i<=k;i++)a[i]=read();
        for(int i=0;i<=17;i++)
        {
            for(int j=1;j<=k;j++)
            {
                int tt=a[j]&(1<<i);
                if(tt==0)edge[0]+=Node{a[j],0ll};
                else 
                {
                    cnt[a[j]]=edge[a[j]].size();
                    edge[a[j]]+=Node{n+1,0ll};
                }
            }
            dijkstra();
//            for(int j=0;j<=n+1;j++)cout<<dis[j]<<" ";
//            cout<<endl;
            ans=min(ans,dis[n+1]);
            for(int j=1;j<=k;j++)
            {
                if(a[j]&(1<<i)){edge[a[j]][cnt[a[j]]].w=INF;}
            }
            edge[0].clear();
            for(int j=1;j<=k;j++)
            {
                int tt=a[j]&(1<<i);
                if(tt==0)
                {  
                    cnt[a[j]]=edge[a[j]].size();
                    edge[a[j]]+=Node{0ll,0ll};
                }
                else edge[n+1]+=Node{a[j],0ll};
            }
            dijkstra2();
            for(int j=1;j<=k;j++)
            {
                if((a[j]&(1<<i))==0){edge[a[j]][cnt[a[j]]].w=INF;}
            }
            edge[n+1].clear();
            ans=min(ans,dis[0]);
        }
        cout<<ans<<endl;
        for(int i=0;i<=n+1;i++)
        {
            edge[i].clear();
         } 
    }
    return 0;
}
```

### 解法二

```cpp
//luoguP5304
//解法二：两遍 dij&多源染色 
#include<cstdio>
#include<iostream>
#include<cstring>
#include<queue>
#define int long long
using namespace std;
const int maxn=1e5+10;
const int maxm=5e5+10;
int U[maxm],V[maxm],W[maxm],f[maxn];
int col1[maxn],col2[maxn],vis1[maxn],vis2[maxn];
int dis1[maxn],dis2[maxn];
int n,m,k;

struct Node{
    int v,w;
    bool operator<(const Node &t)const
    {
        return w>t.w;
    }
};

basic_string<Node>edge[maxn],edge2[maxn];

inline int read()
{
    int x=0,f=1;char ch=getchar();
    while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
    while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
    return x*f;
 } 

void dijkstra()
{
    priority_queue<Node>q;
    memset(vis1,0,sizeof(vis1));
    for(int i=1;i<=n;i++)dis1[i]=(1ll<<50)-1;
    for(int i=1;i<=k;i++)
    {
        q.push(Node{f[i],0ll});
        dis1[f[i]]=0;
        col1[f[i]]=f[i];
//        vis1[f[i]]++;
    }
    while(!q.empty())
    {
        Node now=q.top();
        q.pop();
        if(vis1[now.v])continue;
        vis1[now.v]++;
        for(Node y:edge[now.v])
        {
            if(dis1[y.v]>dis1[now.v]+y.w)
            {
                dis1[y.v]=dis1[now.v]+y.w;
                col1[y.v]=col1[now.v];
                q.push(Node{y.v,dis1[y.v]});
            }
        }
    }
}

void dijkstra2()
{
    priority_queue<Node>q;
    memset(vis2,0,sizeof(vis2));
    for(int i=1;i<=n;i++)dis2[i]=(1ll<<50)-1;
    for(int i=1;i<=k;i++)
    {
        q.push(Node{f[i],0ll});
        dis2[f[i]]=0;
        col2[f[i]]=f[i];
//        vis1[f[i]]++;
    }
    while(!q.empty())
    {
        Node now=q.top();
        q.pop();
        if(vis2[now.v])continue;
        vis2[now.v]++;
        for(Node y:edge2[now.v])
        {
            if(dis2[y.v]>dis2[now.v]+y.w)
            {
                dis2[y.v]=dis2[now.v]+y.w;
                col2[y.v]=col2[now.v];
                q.push(Node{y.v,dis2[y.v]});
            }
        }
    }
}

signed main()
{
    int T;
    T=read();
    while(T--)
    {
        n=read();m=read();k=read();
        for(int i=1;i<=m;i++)
        {
            U[i]=read();V[i]=read();W[i]=read();
            edge[U[i]]+=Node{V[i],W[i]};
        }
        for(int i=1;i<=k;i++)f[i]=read();
        dijkstra();
        for(int i=1;i<=m;i++)edge2[V[i]]+=Node{U[i],W[i]};
        dijkstra2();
        int ans=2e18;
        for(int i=1;i<=m;i++)
        {
            if(col1[U[i]]&&col2[V[i]]&&col1[U[i]]!=col2[V[i]])ans=min(ans,dis1[U[i]]+dis2[V[i]]+W[i]);
         }
        cout<<ans<<'\n';
        for(int i=1;i<=m;i++)edge[U[i]].clear(),edge2[V[i]].clear(); 
    }
    return 0;
}
```

事实上这里的拓展就是今天，~~奥不昨天，已经十二点了~~模拟赛的题，但我甚至都没有想到要用最短路，，还是太菜了。
