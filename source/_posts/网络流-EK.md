---
title: 网络流--EK
date: 2023-10-03 21:07:30
tags: [network-flow,c++]

---

# 网络流——最大流算法EK

###### Date:2023.1.9

## 简介

**EK**算法，即**Edmonds-Karp**动能算法，是求解最大流的基本算法。

## 反向边

**反向边**是帮助程序能发现更优解

例：

---->A---->B---->

发现了一条**增广路**

但后面发现：

---->C---->B---->

---->A---->D---->

这样比原方案更优

所以要建立一条**反向边**，B---->A，让流过去的水能够流回来

下面讲一下建**反向边**方式：

### 链式前向星（链表）

在添加**正向边**时，也把**反向边**给添加（容量为0）。

这样对于第i条边的**反向边**是i\oplus 1（编号请从偶数开始存，~~虽然奇数也不会错（玄学）~~）

### 邻接矩阵

~~这难道还要我说？~~

## 过程

整体来说，就是一个不断**bfs+增广**的过程

为了方便讲解，我们在此定义一些变量：

s：源点,t：汇点，ans：最大流

c：边的容量

f：边的流量

l：点的流量

### bfs

每次从源点出发，进行搜索，若寻找到t，结束**bfs**进行增广。

假设现在我们要从x->u

1.先判断能不能，即$u$是第一次搜到 且 该边的$c>f$

---

2.更新$l_u$，不难得出，$l_u=min(l_x,c-f)$

---

3.记录该边

注意，此时不能直接更新流量，因为不确定该增广路的最大可流流量。

所以要记录该边，等发现增广路后再反跑更新。

---

4.遇到$t$后，跳出**bfs**（找到一条增广路就跳出）

### 增广

反跑增广路

经过$l$在**bfs**里不断刷新，$l_t$即为该增广路的最大可流流量

所以正向边流量+$l_t$

但因为反向边容量为$0$，我们把反向边的容量记为负的（实际是正的，但不影响结果）

即反向边流量-$l_t$

$ans+=l_t$

### 最后

在我们不断跑**bfs、增广、bfs、增广......**

若一次**bfs**后，$l_t=0$，说明**增广路**已被找完，跳出循环

时间复杂度：$\color{orange}O(nm^2)$

```cpp
#include<bits/stdc++.h>
#define M 120005
#define N 1205
using namespace std;
int n,m,s,t;
int tot=1,to[M*8],nt[M*8],tmp[M*4];
int pre[N*2],ppre[N*2];
long long ans,f[M*8],c[M*8];
int qe[N*2],head,tail;
int l[N*2];
long long min(long long aa,long long bb){
    return aa<bb?aa:bb;
}
void add(int x,int y,int u,int uu){
    to[++tot]=y;
    nt[tot]=tmp[x];
    tmp[x]=tot;
    c[tot]=u;
    f[tot]=uu;
}
int main(){
    //freopen("ek.in","r",stdin);
    //freopen("","w",stdout);
    scanf("%d%d%d%d",&n,&m,&s,&t);
    int x,y,z;
    for(register int i=1;i<=m;i++){
        scanf("%d%d%d",&x,&y,&z);
        add(x,y,z,0);
        add(y,x,0,0);
    }
    while(1){
        memset(l,0,sizeof(l));
        memset(pre,0,sizeof(pre));
        memset(ppre,0,sizeof(ppre));
        head=tail=0;
        qe[++tail]=s;
        l[s]=0x3f3f3f3f;
        while(head<tail){
            int u=qe[++head];
            for(register int i=tmp[u];i;i=nt[i]){
                int uu=to[i];
                if(!l[uu]&&c[i]>f[i]){
                    pre[uu]=u,ppre[uu]=i;
                    l[uu]=min(l[u],c[i]-f[i]);
                    qe[++tail]=uu;
                }
            }
            if(l[t]) break;
        }    
        if(!l[t]) break;
        for(register int i=t;i!=s;i=pre[i]){
            f[ppre[i]]+=l[t];
            f[ppre[i]^1]-=l[t];
        }
        ans+=l[t];
    }
    printf("%lld\n",ans);
    return 0;
}
```

参考：[oi-wiki](https://oi-wiki.org//graph/flow/max-flow/)
