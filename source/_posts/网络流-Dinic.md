---
title: 网络流--Dinic
date: 2023-10-04 13:21:39
tags: [network-flow,c++]
mathjax: true
---

# 网络流——最大流算法dinic

###### Date:2023.1.9

## 过程

整体来说，**dinic** 是不断 **bfs分层+dfs增广** 的过程

### bfs分层

每个点的 **层数** 即为它到源点的最短距离，源点的 **层数** 为 0

通过它，我们可以发现：

1.当汇点没有 **层数** 时，算法结束

2.当每次搜索只搜比当前点 **层数** 多一的点，我们找到的肯定是最短增广路

至于怎么分......

~~不会有人不会吧~~

### dfs增广

先看一些变量：

$s：源点,t：汇点，ans：最大流$

$c：边的容量$

$f：边的流量$

$sum：一次dfs的流$

运用分层，我们可以轻易的找到最短增广路

从起点开始，每次往下进行搜索，只搜比当前 **层数** 多一的点，且管道的 $c>f$

当搜到 t 点，结束 **dfs** ，继续分层

### 两个优化

#### 1.多路增广

该优化可让你一次搜到多条增广路

##### 思路

当你从某个点回退到上一个点时，若该点 $流量>0且还有其他可走路径$

那我们就考虑对其再进行 dfs

#### 2.当前弧优化

该优化可帮你避免不必要的搜索

##### 思路

我们知道，当一条边 $c=f$ 时，他就没有用了

这时，我们可以考虑避免对这些边的搜索，把头指针指向下一条边就可以了

同时，对于一次 dfs，它搜过的边不会再搜第二次，我们运用这个更新指针数组

<font color=red>**bfs** 要更新指针数组</font>

时间复杂度：$\color{orange}O\left(N^2M\right)$（运用两个优化后）

特别说明：该复杂度是理论上限，一般会更小

```cpp
#include<bits/stdc++.h>
#define min(x,y) (x<y?x:y)
using namespace std;
const int M=120005,N=1205;
int n,m,s,t;
int tot,to[N*4],nt[N*4],tmp[N*2];
long long f[N*4],c[N*4];
int cur[M*4];
void add(int x,int y,int u,int uu){
    to[++tot]=y;
    nt[tot]=tmp[x];
    tmp[x]=tot;
    c[tot]=u;
    f[tot]=uu;
}
int qe[N*2],head,tail,dep[N];
bool vis[N];
bool bfs(){
    for(register int i=1;i<=N;i++) vis[i]=0; 
    head=tail=0;
    dep[s]=0;
    qe[++tail]=s;
    vis[s]=1;
    cur[s]=tmp[s];
    while(head<tail){
        int x=qe[++head];
        for(register int i=tmp[x];i;i=nt[i]){
            int u=to[i];
            if(!vis[u]&&c[i]>f[i]){
                dep[u]=dep[x]+1;
                cur[u]=tmp[u];
                vis[u]=1;
                qe[++tail]=u;
            }
        }
    }
    return vis[t];
}
long long dfs(int x,int l){
    if(x==t||l==0) return l;
    long long sum=0,ll=0;
    for(register int i=cur[x];i;i=nt[i]){
        int u=to[i];
        cur[x]=i;
        if(dep[x]+1==dep[u]&&c[i]>f[i]){
            ll=dfs(u,min(l,c[i]-f[i]));
            if(ll>0){
                f[i]+=ll,f[i^1]-=ll;
                sum+=ll,l-=ll;
                if(l==0) break;    
            } 
        }
    }
    return sum;
}
long long ans=0;
int main(){
    //freopen("dinic.in","r",stdin);
    //freopen("dinic.out","w",stdout);
    scanf("%d%d",&n,&m);
    s=1,t=m;
    int x,y,z;
    for(register int i=1;i<=n;i++){
        scanf("%d%d%d",&x,&y,&z);
        if(x!=y){
            add(x,y,z,0);
            add(y,x,0,0);
        }
    }
    while(bfs()) ans=ans+dfs(s,0x3f3f3f3f);
    printf("%lld",ans);
    return 0;
}
```

参考：[oi-wiki](https://oi-wiki.org//graph/flow/max-flow/)
