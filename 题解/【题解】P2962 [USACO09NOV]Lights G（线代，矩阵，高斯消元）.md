# 【题解】P2962 [USACO09NOV]Lights G

---

## 题目链接：[P2962 [USACO09NOV]Lights G](https://www.luogu.com.cn/problem/P2962)

## 思路分析：

首先根据题意可以对每一个节点建立一个异或方程组，显然每个节点的状态只由它自己和它相邻节点的状态决定。而我们最终要达到的状态是 1，所以对于每个节点 $i$ ，设它的相邻节点的状态为 $a_1,a_2,a_3,…,a_k$，它自己的状态为 $b_i$，那么 $a_1 \oplus a_2 \oplus a_3 \oplus … \oplus a_k=b_i$ 。

我们可以通过高斯消元解异或方程组，然后解出来一个矩阵。

考虑这个矩阵所满足的特征：

1. 这个矩阵的**主对角线以下的所有位置**均为 0，因为在高斯消元的时候肯定已经通过主对角线将其以下的不为 0 的值都消成 0 了。

2. **主对角线上的位置**有可能为 0，也可能为 1，因为有可能存在无解或多解的情况。

3. **主对角线以上的位置**可能为 0，也可能为 1。当第 $i$ 列上的主对角线上的位置为 1 时，这一列上的其它位置一定为 0（显然主对角线会把他们消为 0）；当第 $i$ 列上主对角线的位置的值是 0，这一列上才有其它位置上的值为 1。

所以可以知道，在经过高斯消元之后得到的矩阵是**上三角矩阵**。

这里引入两个概念：

1. **主元**：主对角线上位置上的值不为 0 的未知量。

2. **自由元**：主对角线位置上的值为 0 的未知量。

可以知道，主元的解是确定的，而自由元的解是不确定的。

因此，我们可以 dfs 暴力枚举所有自由元的解。

根据异或方程组可以知道，要确定每一个点的状态，就要确定与它相连的点的状态，又由于主对角线以下的点上的值一定为 0，所以对这个点的状态可能产生影响的只有编号大于它本身的点。

所以我们可以从下往上搜：判断 $a_{i,i}$ 的值是否为 0，也就是判断它是自由元还是主元：

1. 如果是主元，那么它的解已经可以确定，直接根据异或方程算出它对应的解并累加到答案中即可。

2. 而对于自由元，枚举它的状态是 0 还是 1，并 dfs 即可。

## 易错点：

高斯消元第一层循环结束后 f=false 应该直接 continue 掉而不需要继续搜索。

## 代码实现：

```cpp
//luoguP2962
#include<iostream>
#include<cstdio>
#define int long long
using namespace std;
const int maxn=605;
int a[maxn][maxn],ans=1e18,c[maxn];
int n,m;

inline int read()
{
    int x=0,f=1;char ch=getchar();
    while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
    while(ch>='0'&&ch<='9'){x=x*10+ch-48;ch=getchar();}
    return x*f;
}

bool gauss()
{
    bool f;
    for(int i=1;i<=n;i++)
    {
        f=false;
        for(int j=i;j<=n;j++)
        {
            if(a[j][i])
            {
                f=true;
                for(int k=1;k<=n+1;k++)swap(a[j][k],a[i][k]);
                break;
            }
        }
        if(f==false)continue;//注意这里直接continue即可。
        for(int j=1;j<=n;j++)
        {
            if(j==i||a[j][i]==0)continue;
            for(int k=i+1;k<=n+1;k++)a[j][k]^=a[i][k];//
            a[j][i]=0;
         } 
    }
    return f;
}

void dfs(int now,int pos)
{
    if(now>=ans)return ;//最优性剪枝
    if(pos==0){ans=now;return ;}//更新答案。
    if(a[pos][pos])//主元
    {
        int k=a[pos][n+1];
        for(int i=pos+1;i<=n;i++)if(a[pos][i])k^=c[i];//直接根据异或方程计算答案。
        dfs(now+k,pos-1);
     } 
    else //自由元，暴力枚举是 0 还是 1
    {
        dfs(now,pos-1);
        c[pos]=1;
        dfs(now+1,pos-1);
        c[pos]=0; 
    }
}

signed main()
{
    n=read();m=read();
    for(int i=1;i<=m;i++)
    {
        int u=read(),v=read();
        a[u][v]=a[v][u]=1;//注意要连两条边 
    }
    //自己也会对自己的状态产生影响；将最后一行设为亮着的； 
    for(int i=1;i<=n;i++)a[i][i]=a[i][n+1]=1;
    bool flag=gauss();//判断结果矩阵是否含有自由元（是否为唯一解）
    if(flag)
    {
        int ans=0;
        for(int i=1;i<=n;i++)ans+=a[i][n+1];//答案唯一直接累加求和
        cout<<ans<<endl;
    }
    else dfs(0,n),cout<<ans<<endl;//否则dfs，注意要从后往前搜。
    return 0;
}
```
