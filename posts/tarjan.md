---
title: "浅谈 Tarjan"
desc: "Tarjan 是一种用于解决有向图的连通性及其延伸问题的算法。"
cover: /assets/posts/tarjan.jpg
date: 2023.08.01
tags:
  - 算法
  - C++
  - OI
---

## 强连通分量

有向图 G 强连通是指，G 中任意两个结点连通。

强连通分量（Strongly Connected Components，SCC）的定义是：极大的强连通子图。

## 原理

Tarjan 算法是一种基于 DFS 生成树的回溯算法。在递归搜索图的过程中，Tarjan 主要维护了以下两个变量：

- `dfn[i]` ：时间戳，即按 DFS 序遍历到节点 $i$ 的次序；
- `low[i]` ：在以节点 $i$ 为根的子树中，能够回溯到且仍在栈中的节点的最小时间戳。

其中，在搜索过程中，设当前节点为 $u$ ，会遇到如下情况：

- $u$ 未被访问过：搜索 $u$ 的子树，并用 `low[u]` 更新 `low[i]` ；
- $u$ 已经被访问过，但仍在栈中：用 `dfn[u]` 更新 `low[i]` ；
- $u$ 已经被访问过，且不在栈中：说明 $u$ 所在强连通分量已经搜索完毕，无需做任何操作。

显然，当 `low[i] = dfn[i]` 时，存在一个以 $i$ 为根的强连通分量。我们将 $i$ 及在 $i$ 之后入栈的节点依次出栈并记录，此时这些节点所构成的强连通图即为原图 $G$ 的一个强连通分量。

## 实现

```cpp
int low[N], dfn[N], color[N], idx = 0, sccCnt = 0;
stack<int> s;
set<int> scc[N];

void tarjan(int u) {
    low[u] = dfn[u] = ++idx;
    s.push(u);
    for (auto v : G[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (!color[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        sccCnt++;
        while (!s.empty()) {
            int x = s.top();
            s.pop();
            color[x] = sccCnt;
            scc[sccCnt].insert(x);
            if (x == u) break;
        }
    }
}

int main() {
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) tarjan(i);
    }
    return 0;
}
```

其中，`color[i]` 表示节点 $i$ 所在强连通分量编号，`sccCnt` 表示强连通分量个数。

set `scc[i]` 中存储了编号为 $i$ 的强连通分量所包含的节点。

## 缩点与重新构图

在一些题目中，存在着由多个节点构成的环或连通图。由于强连通分量内每个节点互相可达，我们常常需要使用 Tarjan 或类似算法将这些强连通分量缩为一个点以便使用其他算法求解。这个过程就是**缩点**。

### P2341 受欢迎的牛

题目链接：<https://www.luogu.com.cn/problem/P2341>

在这道题中，奶牛之间的关系显然可以化为有向图求解。显然，在这个有向图中，每个强连通分量里的奶牛都互相喜欢。那么，我们可以对其进行缩点操作。

显然，如果存在明星奶牛，一定存在一个强连通分量，使得该强连通分量缩点后的节点出度为 $0$ 。明星奶牛的个数也就是该强连通分量中的节点数量。

需要注意的是，在缩点后的图中，若存在 $\geq 2$ 个满足上述条件的强连通分量，那么这个图中不存在任何一个明星奶牛，因为无法满足所有其他的奶牛都喜欢该强连通分量中的奶牛。

代码实现：

```cpp
//  P2341 [USACO03FALL / HAOI2006] 受欢迎的牛 G

#include<bits/stdc++.h>

using namespace std;
const int N = 1e4 + 10, M = 5e4 + 10;

int n, m;

class Graph {
private:
    vector<int> G[N];

public:
    int outDegree[N];

    void addEdge(int u, int v) {
        this->G[u].push_back(v);
        this->outDegree[u]++;
    }

    vector<int> operator[] (int idx) {
        return this->G[idx];
    }
} G1, G2;

int low[N], dfn[N], color[N], idx = 0, sccCnt = 0;
stack<int> s;
set<int> scc[N];

void tarjan(int u, Graph& G) {
    low[u] = dfn[u] = ++idx;
    s.push(u);
    for (auto v : G[u]) {
        if (!dfn[v]) {
            tarjan(v, G);
            low[u] = min(low[u], low[v]);
        } else if (!color[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        sccCnt++;
        while (!s.empty()) {
            int x = s.top();
            s.pop();
            color[x] = sccCnt;
            scc[sccCnt].insert(x);
            if (x == u) break;
        }
    }
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        int u, v;
        scanf("%d%d", &u, &v);
        G1.addEdge(u, v);
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) tarjan(i, G1);
    }
    for (int u = 1; u <= n; u++) {
        for (auto v : G1[u]) {
            if (color[u] != color[v]) {
                G2.addEdge(color[u], color[v]);
            }
        }
    }
    long long ans = 0;
    for (int i = 1; i <= sccCnt; i++) {
        if (G2.outDegree[i] == 0) {
            if (!ans) ans = scc[i].size();
            else {
                printf("0\n");
                return 0;
            }
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

