---
title: AC 自动机
date: 2024-03-09 16:20:15
tags: [c++,string]
categories: [algorithm]
mathjax: true
---

# AC 自动机

> 很多人在第一次看到这个东西的时侯是非常兴奋的。不过这个自动机叫作 `Automaton`，不是 `Automation`，这里的 `AC` 也不是 `Accepted`，而是 `Aho–Corasick`（Alfred V. Aho, Margaret J. Corasick. 1975）。

## 前置

Trie （字典树）

KMP （只要失配指针的思想即可）

## 定义

AC 自动机是 **以 Trie (字典树) 的结构为基础**，结合 **KMP (字符串匹配) 的思想** 建立的自动机，用于解决多模式匹配等任务。

## 构建

先简单解释一下：

AC 自动机由两部分组成：

1. 将模式串构造成一棵普通的 Trie。

2. 利用 KMP 思想构造失配指针。

### 失配指针

同 KMP 一样，AC 自动机同样构造一个 fail 用于失配时的跳转指针。

但 AC 自动机的 fail 指向当前状态的最长后缀。

定义如下变量：

当前节点 `u`，`u` 的父亲节点 `p`，`p` 和 `u` 间用一条字符 `c` 连接。 

字典树的边集 `tr`。（$tr_{u,c}$ 表示节点 `u` 的 `c` 出边的点）

假设所有深度小于 `u` 的 fail 指针均已求得。

1. 若 $tr_{fail_{p}, c}$ 存在，则让 $fail_u=tr_{fail_{p}, c}$。

2. 否则，找到 $fail_{fail_p}$，重复 1 的过程，直到根节点。

3. 如果还没有，令 $fial_u=root$。（$root$ 表示根节点）

### 自动机

AC 自动机通过 **一个** BFS 函数实现 fail 和 自动机 的构建。

定义如下变量：

字典树的边集 `tr`。（$tr_{u,c}$ 表示节点 `u` 的 `c` 出边的点）

1. 将根节点的所有子节点入队。

2. 对于队头 `u`，遍历字符集，设当前遍历到 `i`。
   
   1. 若 $tr_{u,i}$ 存在，则将 $fail_{tr_{u,i}}=tr_{fail_u,i}$。
   
   2. 否则，令 $tr_{u,i}=tr_{fail_u,i}$。

```cpp
queue<int> q;
void build() {
    for (int i = 0; i < 26; i++)
        if (nd[0].son[i]) q.push(nd[0].son[i]);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int i = 0; i < 26; i++)
            if (nd[u].son[i])
                nd[nd[u].son[i]].fail = nd[nd[u].fail].son[i], q.push(nd[u].son[i]);
            else
                nd[u].son[i] = nd[nd[u].fail].son[i];
    } 
}
```

### 匹配

遍历文本串 `T`，设当前点为 `u`：

1. 利用 fail 找到所有匹配的模式串，累加，清零。

```cpp
int query(char *t) {
        int u = 0, sum = 0;
        for (int i = 1; t[i]; i++) {
            int c = t[i] - 'a';
            u = nd[u].son[c];
            for (int j = u; j && nd[j].e != -1; j = nd[j].fail)
                sum += nd[j].e, nd[j].e = -1;
        }
        return sum;
    }
```

### 例题

#### [P3808 AC 自动机（简单版）](https://www.luogu.com.cn/problem/P3808)

直接套 AC 自动机。

```cpp
#include <queue>
#include <stdio.h>
using namespace std;
const int N = 1e6 + 5;
int n;
struct AC {
    int cnt;
    struct node {
        int son[30], fail, e;
    } nd[N];
    void insert(char *s) {
        int u = 0;
        for (int i = 1; s[i]; i++) {
            int c = s[i] - 'a';
            if (!nd[u].son[c]) nd[u].son[c] = ++cnt;
            u = nd[u].son[c];
        }
        nd[u].e++;
    }
    queue<int> q;
    void build() {
        for (int i = 0; i < 26; i++)
            if (nd[0].son[i]) q.push(nd[0].son[i]);
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int i = 0; i < 26; i++)
                if (nd[u].son[i])
                    nd[nd[u].son[i]].fail = nd[nd[u].fail].son[i], q.push(nd[u].son[i]);
                else
                    nd[u].son[i] = nd[nd[u].fail].son[i];
        }
    }
    int query(char *t) {
        int u = 0, sum = 0;
        for (int i = 1; t[i]; i++) {
            int c = t[i] - 'a';
            u = nd[u].son[c];
            for (int j = u; j && nd[j].e != -1; j = nd[j].fail)
                sum += nd[j].e, nd[j].e = -1;
        }
        return sum;
    }
} ac;
char s[N];
int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%s", s + 1), ac.insert(s);
    ac.build();
    scanf("%s", s + 1);
    printf("%d\n", ac.query(s));
    return 0;
}
```

#### [P3796 AC 自动机（简单版 II）](https://www.luogu.com.cn/problem/P3796)

基本思路一样。

多记录两个东西：

1. 每个模式串在字典树中结束的位置

2. 每个字典树上的点匹配到模式串的次数

```cpp
#include <cstring>
#include <queue>
#include <stdio.h>
using namespace std;
const int N = 156, T = 1e6 + 6, S = N * 80;
struct AC {
    int cnt, ans[N];
    struct node {
        int son[30], fail, idx, val;
    } nd[S];
    void init() {
        memset(nd, 0, sizeof nd);
        memset(ans, 0, sizeof(ans));
        cnt = 0;
    }
    void insert(char *s, int id) {
        int u = 0;
        for (int i = 1; s[i]; i++) {
            int c = s[i] - 'a';
            if (!nd[u].son[c]) nd[u].son[c] = ++cnt;
            u = nd[u].son[c];
        }
        nd[u].idx = id;
    }
    queue<int> q;
    void build() {
        for (int i = 0; i < 26; i++)
            if (nd[0].son[i]) q.push(nd[0].son[i]);
        while (q.size()) {
            int u = q.front();
            q.pop();
            for (int i = 0; i < 26; i++)
                if (nd[u].son[i])
                    nd[nd[u].son[i]].fail = nd[nd[u].fail].son[i], q.push(nd[u].son[i]);
                else
                    nd[u].son[i] = nd[nd[u].fail].son[i];
        }
    }
    int query(char *t) {
        int u = 0, sum = 0;
        for (int i = 1; t[i]; i++) {
            int c = t[i] - 'a';
            u = nd[u].son[c];
            for (int j = u; j; j = nd[j].fail) nd[j].val++;
        }
        for (int i = 0; i <= cnt; i++)
            if (nd[i].idx) sum = max(sum, nd[i].val), ans[nd[i].idx] = nd[i].val;
        return sum;
    }
} ac;
int n;
char s[N][100], t[T];
int main() {
    while (scanf("%d", &n)) {
        if (n == 0) break;
        for (int i = 1; i <= n; i++)
            scanf("%s", s[i] + 1), ac.insert(s[i], i);
        ac.build();
        scanf("%s", t + 1);
        int x = ac.query(t);
        printf("%d\n", x);
        for (int i = 1; i <= n; i++)
            if (ac.ans[i] == x) printf("%s\n", s[i] + 1);
        ac.init();
    }
    return 0;
}
```

## 优化

在 AC 自动机中，会一直跳 fail 寻找匹配，但这样效率极低。

考虑优化：

先考虑 fail 的特性：一定是一棵树。

而我们 AC 自动机的匹配就可以转化为在 fail 树上的链求和问题。

这里提供两种思路。

例题：[P5357 【模板】AC 自动机](https://www.luogu.com.cn/problem/P5357)

### 拓扑优化（快）

考虑预先记录所有跳的 fail，最后一并求和。

于是按照 fail 树建图。

```cpp
#include <queue>
#include <stdio.h>
using namespace std;
const int N = 2e5 + 5, S = 2e5 + 5, T = 2e6 + 5;
char s[S], t[T];
int n;
struct AC {
    int cnt = 1, ans[N], f[N], in[N];
    struct node {
        int son[27], fail, f, ans;
    } nd[N];

    queue<int> q;

    void insert(char *s, int id) {
        int u = 1;
        for (int i = 1; s[i]; i++) {
            int c = s[i] - 'a';
            if (!nd[u].son[c]) nd[u].son[c] = ++cnt;
            u = nd[u].son[c];
        }
        f[id] = (nd[u].f) ? nd[u].f : nd[u].f = id;
        return;
    }

    void build() {
        for (int i = 0; i < 26; i++) nd[0].son[i] = 1;
        nd[1].fail = 0;
        q.push(1);
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            int fail = nd[u].fail;
            for (int i = 0; i < 26; i++) {
                int v = nd[u].son[i];
                if (!v) {
                    nd[u].son[i] = nd[fail].son[i];
                    continue;
                }
                nd[v].fail = nd[fail].son[i], in[nd[fail].son[i]]++;
                q.push(v);
            }
        }
    }

    void topsort() {
        for (int i = 1; i <= cnt; i++)
            if (!in[i]) q.push(i);
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            ans[nd[u].f] = nd[u].ans;
            int fail = nd[u].fail;
            nd[fail].ans += nd[u].ans;
            if (!--in[fail]) q.push(fail);
        }
    }

    void query(char *s) {
        int u = 1;
        for (int i = 1; s[i]; i++) u = nd[u].son[s[i] - 'a'], nd[u].ans++;
    }
} ac;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%s", s + 1), ac.insert(s, i);
    ac.build();
    scanf("%s", t + 1);
    ac.query(t), ac.topsort();
    for (int i = 1; i <= n; i++) printf("%d\n", ac.ans[ac.f[i]]);
    return 0;
}
```

### 子树求和（慢）

先子树求和，最后累加。

```cpp
#include <deque>
#include <stdio.h>
using namespace std;
typedef deque<int> dq;
const int N = 2e5 + 5, S = 2e5 + 5, T = 2e6 + 5;
int n;
char s[S], t[T];
int cnt[N];
struct ACM {
    struct node {
        int son[30], val, fail, hd;
        dq idx;
    } nd[S];
    struct edge {
        int to, nt;
    } e[S];
    int rt, acnt, tot;
    void insert(char *s, int id) {
        int u = rt;
        for (int i = 1; s[i]; i++) {
            int c = s[i] - 'a' + 1;
            if (!nd[u].son[c]) nd[u].son[c] = ++acnt;
            u = nd[u].son[c];
        }
        nd[u].idx.push_back(id);
    }
    void build() {
        dq q;
        for (int i = 1; i <= 26; i++)
            if (nd[rt].son[i]) q.push_back(nd[rt].son[i]);
        while (!q.empty()) {
            int u = q.front();
            q.pop_front();
            for (int i = 1; i <= 26; i++)
                if (nd[u].son[i]) {
                    nd[nd[u].son[i]].fail = nd[nd[u].fail].son[i];
                    q.push_back(nd[u].son[i]);
                } else
                    nd[u].son[i] = nd[nd[u].fail].son[i];
        }
    }

    void query(char *s) {
        int u = rt;
        for (int i = 1; s[i]; i++) {
            int c = s[i] - 'a' + 1;
            u = nd[u].son[c], nd[u].val++;
        }
    }

    void add(int u, int v) { e[++tot] = {v, nd[u].hd}, nd[u].hd = tot; }

    void dfs(int x) {
        for (int i = nd[x].hd; i; i = e[i].nt) {
            int v = e[i].to;
            dfs(v);
            nd[x].val += nd[v].val;
        }
        for (auto i : nd[x].idx) cnt[i] += nd[x].val;
    }

    void failtr() {
        for (int u = 1; u <= acnt; u++) add(nd[u].fail, u);
        dfs(rt);
    }
} ac;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%s", s + 1);
        ac.insert(s, i);
    }
    ac.build();
    scanf("%s", t + 1);
    ac.query(t), ac.failtr();
    for (int i = 1; i <= n; i++) printf("%d\n", cnt[i]);
    return 0;
}
```

## 后记

这篇文章是近半年未写算法后的第一篇文章，很不容易，希望大家多支持。



参考 : [AC 自动机 - OI Wiki](https://oi-wiki.org/string/ac-automaton/#kmp-%E8%87%AA%E5%8A%A8%E6%9C%BA)
