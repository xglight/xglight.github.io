---
title: 网络流--SAP+GAP
date: 2023-10-03 21:09:15
tags: [network-flow,c++]

---

# 网络流——最大流算法SAP+GAP

###### Date:2023.1.12

## 简介

**SAP**和**dinic**十分相似，应该比较好懂

**GAP**则是**SAP**的优化

## SAP

**SAP**算法（也称**ISAP**），也是运用**bfs分层**+**dfs搜索**

与**dinic**区别有两点：

1.只用了一次**bfs分层**，后续层数更新随着**dfs**一起

2.因为某些原因（**dfs**更新层数），**dfs**需要反跑（汇点->源点）

### 过程

因为**SAP**的**dfs**需要反跑，知道大家肯定打不习惯

于是，我们可以反跑**bfs分层**，就可以正跑**dfs**（反跑时汇点层数为$0$）

**dfs**时，过程与**dinic**基本相同，这里不详细展开

主要说如何更新层数：

#### 更新层数

设$i$点的层数为$d_i$

当我们结束了一次**dfs**（没有路可走），我们就要更新层数

遍历$i$的所有连点$（c>f）$，找到连点中最小的层数，设为$d_j$

令$d_i=d_j+1$

若点$i$没有连点，令$d_i=n$

另外，我们不难发现，当$d_s\ge n$ ，增广路已被搜完，结束循环

#### 优化

当前弧优化

时间复杂度：$\color{orange}O(n^2m)$

## GAP

在**SAP**算法的基础上，我们加一个数组$num$

$num_i$表示层数为$i$的点有多少个

更新层数时同时也更新$num$

若一次更新后，$num_i=0$，说明图中出现断层，不需要再搜索，直接退出

时间复杂度：$\color{orange}十分可观（小于n^2m）$

## 特别说明：

**SAP**的复杂度是$\color{orange}O(n^2m)$

但**GAP**优化较不稳定（但一定小于$\color{orange}n^2m$），给**SAP**的复杂度带来较大的向下浮动

所以，<font color=red>**GAP**和**dinic**哪个更优就十分奇妙了</font>

建议：

小数据可用**dinic**

大数据用**GAP**

当然，也可以打两个，测一下哪个更优

```cpp
#include<bits/stdc++.h>
#define N 1205
#define M 120005
using namespace std;
int n,m,s,t;
int tot=1,to[M*4],nt[M*4],tmp[M*2];
int pre[M*2],ppre[M*2];
int cur[M*2],num[M*2];
long long ans,sum=0x3f3f3f3f3f3f3f3f,f[M*4],c[M*4];
void add(int x,int y,int u,int uu){
    to[++tot]=y;
    nt[tot]=tmp[x];
    tmp[x]=tot;
    c[tot]=u;
    f[tot]=uu;
}
long long min(long long aa,long long bb){
    return aa<bb?aa:bb;
}
int qe[M*2],head,tail;
int dep[M];
bool vis[M],oo[205][205];
void bfs(){
    for(register int i=0;i<=M;i++) vis[i]=0; 
    head=tail=0;
    dep[t]=0;
    qe[++tail]=t;
    vis[t]=1;
    cur[t]=tmp[t];
    num[0]=1;
    while(head<tail){
        int x=qe[++head];
        for(register int i=tmp[x];i;i=nt[i]){
            int u=to[i];
            if(!vis[u]&&c[i^1]>f[i^1]){
                cur[u]=tmp[u];
                dep[u]=dep[x]+1;
                vis[u]=1;
                qe[++tail]=u;
                num[dep[u]]++;
            }
        }
    }
}
int main(){
    //freopen("gap.in","r",stdin); 
    scanf("%d%d%d%d",&n,&m,&s,&t);
    int xx,yy,zz;
    for(register int i=1;i<=m;i++){
        scanf("%d%d%d",&xx,&yy,&zz);
        add(xx,yy,zz,0),add(yy,xx,0,0);
    }
    int x=s;
    bfs();
    for(register int i=1;i<=n;i++) cur[i]=tmp[i];
    while(dep[s]<n){
        if(x==t){
            ans+=sum;
            for(register int i=t;i!=s;i=pre[i]){
                f[ppre[i]]+=sum;
                f[ppre[i]^1]-=sum;
            }
            x=s;
            sum=0x3f3f3f3f3f3f3f3f;
        }
        bool flag=0;
        for(register int i=cur[x];i;i=nt[i]){
            int u=to[i];
            if(c[i]>f[i]&&dep[x]==dep[u]+1){
                flag=1;
                pre[u]=x,ppre[u]=i;
                cur[x]=i;
                sum=min(sum,c[i]-f[i]);
                x=u;
                break;
            }
        }
        if(!flag){
            int y=n-1;
            for(register int i=tmp[x];i;i=nt[i]){
                int u=to[i];
                if(c[i]>f[i]) y=min(y,dep[u]);
            }
            --num[dep[x]];
            dep[x]=y+1;
            num[dep[x]]++;
            if(num[dep[x]]==0) break;
            cur[x]=tmp[x];
            if(x!=s) x=pre[x];
        }
    }
    printf("%lld",ans);
    return 0;
}
```

参考：[oi-wiki](https://oi-wiki.org//graph/flow/max-flow/)
