---
title: "浅谈差分约束"
desc: "差分约束是一种能够求解多个多元不等式组的图论建图构造方法。"
cover: /assets/posts/diff-constraints.jpg
date: 2024.11.16
tags:
  - 算法
  - C++
  - OI
---

## 差分约束

差分约束系统是一种特殊的 $n$ 元一次不等式组，形如：
$$
\left\{
\begin{aligned}
&x_i \leq x_j + c_k,\\
&x_{i'} \leq x_{j'} + c_{k'},\\
&\cdots
\end{aligned}
\right.
$$
注意到这与最短路算法中 $\text{dis}_{v} \leq \text{dis}_{u} + w$ 类似。所以我们考虑构造一种特殊的图，使得这个图能够通过某种途径判断它所代表的差分约束系统是否有解。

> 注：若一个差分约束系统有解，则它必有无数个解。
>
> **证明：**若 ${a_1, a_2, \cdots, a_n}$ 为差分约束系统 $U$ 的一组解，则其必有另一组解 ${a_1 + d, a_2 + d, \cdots, a_n + d}$ ，其中 $d \in \R$ 。故得证。

考虑如何求解。我们显然可以通过新建一条边权为 $c_k$ 的边 $E_{j,i}: j \rarr i$ 来表示不等式 $x_i \leq x_j + c_k$ ，以此类推，将每个变量 $x_u$ 看作图中的一个节点，将每个不等式看作图中的一条带权有向边。

由于通过上述方法构造的图无法保证连通，我们不妨设超级源点 $U = x_0$ ，然后从 $U$ 出发向每个节点 $x_i$ （$x_i > 0$） 连接一条边权为 $0$ 的有向边 $E_{0,i}: 0 \rarr i$ 来保证连通性。之后，我们通过从节点 $U$ 开始求单源最短路并检查负环是否存在。若负环存在，则给定的差分约束系统无解；否则 $x_i = \text{dis}_i$ 即为该差分约束系统的一组可能解。

在实现上我们一般采用 SPFA（队列优化的 Bellman-Ford ）作为最短路算法。注意判断负环时因超级源点的存在而导致节点数相应增加。

## 例题

### [P1993 小 K 的农场](https://www.luogu.com.cn/problem/P1993)

#### 题目大意

给定 $n$ 个约束条件，其中分为三类：

- $x_a - x_b \geq c$
- $x_a - x_b \leq c$
- $x_a = x_b$

给定序列 $x_i$ 和 $n$ 个约束条件，求该差分约束系统是否有解。

#### 题目分析

模板题。考虑对于每个操作进行转化，使其成为形如 $x_i \leq x_j + c_k$ 的形式：

- $x_a - x_b \geq c \Rarr x_b \leq x_a - c$
- $x_a - x_b \leq c \Rarr x_a \leq x_b + c$
- $x_a = x_b \Rarr x_a \leq x_b\ ,\ x_b \leq x_a$

最后建边并连接超级源点、检测负环即可。

#### 代码

```cpp
// Problem: P1993 小 K 的农场
// Contest: Luogu
// URL: https://www.luogu.com.cn/problem/P1993
// Memory Limit: 128 MB
// Time Limit: 1000 ms
//
// Powered by CP Editor (https://cpeditor.org)

#include <memory.h>

#include <algorithm>
#include <cmath>
#include <cstdio>
#include <iostream>
#include <queue>
#include <stack>
#include <string>
#include <vector>
using namespace std;
const int N = 5 * 1e5 + 10;

int n, m;
int h[2 * N], vtx[2 * N], nxt[2 * N], w[2 * N], idx, vis[2 * N], dis[2 * N];
int cnt[2 * N], op, a, b, c;
queue<int> q;

void addEdge(int a, int b, int c) {
    vtx[idx] = b, nxt[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa(int s) {
    memset(dis, 0x3f, sizeof(dis));
    memset(vis, 0, sizeof(vis));
    dis[s] = 0;
    q.push(s);
    vis[s] = 1;
    while (!q.empty()) {
        int tmp = q.front();
        vis[tmp] = 0;
        int p = h[tmp];
        while (p != -1) {
            int v = vtx[p];
            // 此处不能取等，否则会把 0 环判定为负环
            if (dis[tmp] + w[p] < dis[v]) {
                dis[v] = dis[tmp] + w[p];
                cnt[v]++;
                if (cnt[v] > n) return 0;
                if (!vis[v]) {
                    q.push(v);
                    vis[v] = 1;
                }
            }
            p = nxt[p];
        }
        q.pop();
    }
    return 1;
}

int main() {
    memset(h, -1, sizeof(h));
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> op;
        if (op == 1) {
            cin >> a >> b >> c;
            addEdge(a, b, -c);
        } else if (op == 2) {
            cin >> a >> b >> c;
            addEdge(b, a, c);
        } else {
            cin >> a >> b;
            addEdge(b, a, 0);
            addEdge(a, b, 0);
        }
    }
    for (int i = 1; i <= n; i++) {
        addEdge(n + 1, i, 0);
    }
    if (spfa(n + 1)) {
        cout << "Yes" << endl;
    } else {
        cout << "No" << endl;
    }
    return 0;
}
```

### [P2294 [HNOI2005] 狡猾的商人](https://www.luogu.com.cn/problem/P2294)

#### 题目大意

给定 $m$ 条关于前缀和数组 $c_i$ 的信息，每条信息形如 $c_t - c_{s - 1} = v_k$ ，求该差分约束系统是否有解。

#### 题目分析

根据题目抽象出题目大意后问题就再次转化为裸的差分约束问题，对每条信息可转化为：

- $c_t \leq c_{s - 1} + v_k$
- $c_{s - 1} \leq c_t - v_k$

建边跑最短路即可。注意超级源点。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5 + 10;

struct Edge {
    int v, w;
};

vector<Edge> G[N];
int cnt[N], dis[N], vis[N];
int n, m, T;

void addEdge(int u, int v, int w) {
    G[u].push_back(Edge{v, w});
}

bool spfa(int u) {
    queue<int> q;
    memset(dis, 0x3f, sizeof(dis));
    memset(cnt, 0, sizeof(cnt));
    memset(vis, 0, sizeof(vis));
    dis[u] = 0, vis[u] = 1;
    q.push(u);
    while (!q.empty()) {
        int t = q.front();
        q.pop();
        vis[t] = 0;
        for (auto edge : G[t]) {
            int v = edge.v, w = edge.w;
            if (dis[v] > dis[t] + w) {
                dis[v] = dis[t] + w;
                cnt[v] = cnt[t] + 1;
                if (cnt[v] >= n + 2) {
                    return 1;
                }
                if (!vis[v]) {
                    q.push(v);
                    vis[v] = 1;
                }
            }
        }
    }
    return 0;
}

int main() {
    scanf("%d", &T);
    while (T--) {
        for (int i = 0; i < N; i++) {
            G[i].clear();
        }
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= m; i++) {
            int s, t, v;
            scanf("%d%d%d", &s, &t, &v);
            // sum[t] - sum[s - 1] = v
            // sum[t] - sum[s - 1] >= v
            // sum[t] - sum[s - 1] <= v
            // sum[s - 1] + v <= sum[t]
            // sum[t] - v <= sum[s - 1]
            addEdge(s - 1, t, v);
            addEdge(t, s - 1, -v);
        }
        for (int i = 0; i <= n; i++) {
            addEdge(n + 1, i, 0);
        }
        bool flag = spfa(n + 1);
        if (flag) {
            printf("false\n");
        } else {
            printf("true\n");
        }
    }
    return 0;
}
```

### [P4878 [USACO05DEC] Layout G](https://www.luogu.com.cn/problem/P4878)

#### 题目大意

给定一个长度为 $n$ 的数列 $a_i$ 和 $m$ 条信息，对于每条信息，有两种可能：

1. $d_a - d_b \leq c_k$
2. $d_a - d_b \geq c_k$

其中 $d_i$ 表示第 $i$ 个数所在的位置。保证数据有序给出。

求 $(d_n - d_1)_{\text{max}}$ 。

#### 题目分析

一个有点意思的题目。

对于判断是否有解的方法显然，套用差分约束模板即可。在此主要探讨关于求解 $(d_n - d_1)_{\text{max}}$ 。

显然我们无法直接使用由超级源点 $U$ 出发的最短路数组（正权均被设为 $0$ ），不妨在有解的情况下从结点 $1$ 开始跑第二次最短路，此时的 $\text{dis}$ 数组即为 $1$ 出发可到达的节点的最短路，即为差分约束系统的部分可能解。若 $\text{dis}_n = +\infin$ ，则代表从 $1$ 没有相应的条件指向 $n$ ，此时两者可以相距正无穷远（即为输出 `-2` 时的情况）。

这道题还有一个细节需要注意。题目明确，可以有多个奶牛在同一位置上，即两个相邻奶牛最小距离 $\geq 0$ 。即 $d_i - d_{i - 1} \geq 0$ 也需要作为差分约束系统的条件之一参与决策。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e3 + 10;
const int INF = 0x3f3f3f3f;

int n, ml, md;

struct Edge {
    int v, w;
};
vector<Edge> G[N];

int cnt[N], dis[N], vis[N];

void addEdge(int u, int v, int w) {
    G[u].push_back(Edge{v, w});
}

bool spfa(int s) {
    queue<int> q;
    memset(vis, 0, sizeof(vis));
    memset(cnt, 0, sizeof(cnt));
    memset(dis, 0x3f, sizeof(dis));
    dis[s] = 0;
    q.push(s);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        vis[u] = 0;
        for (auto [v, w] : G[u]) {
            if (dis[v] > dis[u] + w) {
                dis[v] = dis[u] + w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n + 1) {
                    return 0;
                }
                if (!vis[v]) {
                    vis[v] = 1;
                    q.push(v);
                }
            }
        }
    }
    return 1;
}

int main() {
    scanf("%d%d%d", &n, &ml, &md);
    for (int i = 1; i <= ml; i++) {
        int a, b, d;
        scanf("%d%d%d", &a, &b, &d);
        // s_b - s_a <= d
        // s_b <= s_a + d
        addEdge(a, b, d);
    }
    for (int i = 1; i <= md; i++) {
        int a, b, d;
        scanf("%d%d%d", &a, &b, &d);
        // s_b - s_a >= d
        // s_a <= s_b - d
        addEdge(b, a, -d);
    }
    for (int i = 2; i <= n; i++) {
        // s_i - s_i-1 >= 0
        // s_i-1 <= s_i - 0
        addEdge(i, i - 1, 0);
    }
    for (int i = 1; i <= n; i++) {
        addEdge(0, i, 0);
    }
    bool flag = spfa(0);
    if (flag) {
        spfa(1);
        if (dis[n] == INF) {
            printf("-2\n");
        } else {
            printf("%d\n", dis[n]);
        }
    } else {
        printf("-1\n");
    }
    return 0;
}
```

