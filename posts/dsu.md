---
title: "浅谈并查集"
desc: "并查集是一种用于管理元素所属集合的数据结构。"
cover: /assets/posts/dsu.jpg
date: 2024.11.17
tags:
  - 算法
  - C++
  - OI
---

## 并查集

并查集是一种用于管理元素所属集合的数据结构，

并查集支持两种操作：

- 合并（`merge`）：合并两个节点所属的结合（合并两个树）
- 查询（`find`）：查询某个节点所对应的集合

使用路径压缩后的并查集的平均时间复杂度为 $O(\alpha(n))$ ，其中 $\alpha(n)$ 表示阿克曼函数的反函数。在宇宙可观测的 $n$ 中，$\alpha(n) \leq 5$ ，即路径压缩后的平均时间复杂度为渐进 $O(1)$ 。但是，可以构造特殊数据使得朴素路径压缩（没有进行按秩合并）的并查集实现达到单次查询 $O(\log n)$ 的最坏时间复杂度。具体证明和验证见 <https://leetcode.cn/problems/number-of-provinces/solutions/550060/jie-zhe-ge-wen-ti-ke-pu-yi-xia-bing-cha-0unne/> 以及 <https://oi-wiki.org/ds/dsu-complexity/#%E4%B8%BA%E4%BD%95%E5%B9%B6%E6%9F%A5%E9%9B%86%E4%BC%9A%E8%A2%AB%E5%8D%A1> 。一般情况下可以认为并查集实现为均摊 $O(1)$ 。

若追求严格 $O(1)$ ，可以通过按秩合并或启发式合并实现最坏 $O(\alpha(n))$ 即均摊 $O(1)$ 的时间复杂度。详见 <https://oi-wiki.org/ds/dsu/#%E5%90%88%E5%B9%B6> 。

本文将仅采用路径压缩的朴素并查集。下面给出面向对象的模板代码：

```cpp
class DSU {
private:
    int fa[N];

public:
    DSU() {
        for (int i = 0; i < N; i++) {
            fa[i] = i;
        }
    }

    int find(int x) {
        if (fa[x] == x) return x;
        return fa[x] = find(fa[x]);
    }

    void merge(int u, int v) {
        int fu = find(u), fv = find(v);
        if (fu != fv) {
            fa[fu] = fv;
        }
    }
};
```

注意 `fa[i]` 数组的初始化赋值。

## 例题

### [P2024 [NOI2001] 食物链](https://www.luogu.com.cn/problem/P2024)

#### 题目大意

有 A 吃 B 、B 吃 C 、C 吃 A 三种狩猎关系，保证给出的每个动物都是 A, B, C 中的一种，判断对于每条给定信息是否与先前信息矛盾。输出矛盾信息的总数。

每条信息分为两类：

1. 给定 X 和 Y ，表示 X 和 Y 是同类；
2. 给定 X 和 Y ，表示 X 吃 Y 。

#### 题目分析

显然可以通过种类并查集暴力维护，具体实现方法不在此赘述。此处将主要探讨带权并查集的解法。

不妨对 X, Y 可能产生的所有关系进行枚举。规定如下权值：

- `0` 表示此节点与父节点为同类；
- `1` 表示此节点被父节点吃；
- `2` 表示此节点吃父节点。

随后对每个可能的合法关系进行枚举，得到下表：

| 节点 X 与父关系 | 节点 X 与 Y 关系 |  节点 Y 与父关系|
| :----------: | :----------: | :----------: |
|0  |  0|  0|
|0  | 1 |  1|
|  0| 2 |  2|
|  1|  0|  1|
|  1|  1|  2|
|  1|  2|  0|
|  2|  0|  2|
|  2|  1|  0|
|  2|  2|  1|

设 $w_i$ 表示节点 $i$ 与其父节点的关系。观察规律可得，有 $w_Y = (w_{X,Y}' + w_X) \bmod 3$ 。

其中 $w_{X,Y}'$ 表示 X 与 Y 之间的关系。上式可用于维护并查集的合并操作。

设 F1 , F2 分别为 X 和 Y 的父节点（根），则在更新时有 $w_{F1} = w'_{X,Y} + w_Y - w_X$ 。具体证明可以画图易得。

注意实现时需要注意溢出问题，即 $w_{F1} = (w'_{X,Y} + w_Y - w_X + 3) \bmod 3$ 。

查找时我们需要对并查集进行路径压缩。显然此时我们可以通过 $w_Y'' = (w_Y + w_X) \bmod 3$ ，其中 $w''_Y$ 表示更新后的权值，来更新每一个路径。

对于判断每条信息是否矛盾，我们首先需要判断 F1 与 F2 是否在一个集合中。若不在一个集合中，则表明 X 和 Y 之间没有明确的信息表示它们之间的关系，故不矛盾；否则我们需要分情况讨论：

- 若当前信息为 X 和 Y 是同类，则不矛盾时的情况由上表可知需要满足 $w_X = w_Y$ ，故若 $w_X \neq w_Y$ 时信息矛盾；
- 若当前信息为 X 吃 Y ，则不矛盾时的情况由上表可知需要满足 $w_Y - w_X = 1$ 即 $(w_Y - w_X + 3) \bmod 3 = 1$ 时不矛盾，反之则当前信息矛盾。

至此我们就完成了这道题目主体的分析。可以发现，我们事实上通过该并查集维护了一个在 $\bmod 3$ 意义下的整数加法群。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5 + 10;

int n, m, ans = 0;
int fa[N], w[N];
/**
 * 0 = SAME
 * 1 = PREDATOR
 * 2 = PREY
 */

int find(int x) {
    if (fa[x] == x) return x;
    int rFa = find(fa[x]);
    w[x] = (w[x] + w[fa[x]]) % 3;
    return fa[x] = rFa;
}

void merge(int u, int v, int weight) {
    int fu = find(u), fv = find(v);
    if (fu != fv) {
        fa[fu] = fv;
        /**
         * fu ---> fv
         * ↑       ↑
         * u - - > v
         */
        w[fu] = (weight + w[v] - w[u] + 3) % 3;
    }
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) fa[i] = i;
    for (int i = 1; i <= m; i++) {
        int op, x, y;
        scanf("%d%d%d", &op, &x, &y);
        if (x > n || y > n) {
            ans++;
            continue;
        }
        if (op == 1) {
            int fx = find(x), fy = find(y);
            if (fx == fy && w[x] != w[y]) {
                ans++;
            } else {
                merge(x, y, 0);
            }
        } else {
            int fx = find(x), fy = find(y);
            if (fx == fy && (w[x] - w[y] + 3) % 3 != 1) {
                ans++;
            } else {
                merge(x, y, 1);
            }
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

### [P1196 [NOI2002] 银河英雄传说](https://www.luogu.com.cn/problem/P1196)

#### 题目大意

在 $[1, 3 \times 10^4]$ 上有分布不同的数列，定义合并操作为将第 $j$ 列的数列的列首接到第 $i$ 列的列尾。

给定 $T$ 个操作，每个操作分为两类：

- 合并操作，将第 $i$ 列的数列合并到第 $j$ 列之后；
- 查询操作，查询编号为 $i$ 的数和 $j$ 的数是否在同一列中，若在同一列中求 $i$ 和 $j$ 之间有多少个数间隔。

保证 $i \neq j$ 。

#### 题目分析

带权并查集。考虑维护每个数所在队列的总长度 $\text{cnt}_i$ 和其到当前队首的距离 $\text{w}_i$ 。

直接维护即可。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 30010;

int T;
int fa[N], w[N], cnt[N];

int find(int x) {
    if (fa[x] == x) return x;
    int rFa = find(fa[x]);
    w[x] += w[fa[x]];
    cnt[x] = cnt[fa[x]];
    return fa[x] = rFa;
}

void merge(int u, int v, int weight) {
    int fu = find(u), fv = find(v);
    if (fu != fv) {
        fa[fu] = fv;
        w[fu] += cnt[fv];
        cnt[fu] += cnt[fv];
        cnt[fv] = cnt[fu];
    }
}

int main() {
    cin >> T;
    for (int i = 1; i < N; i++) {
        fa[i] = i;
        cnt[i] = 1;
    }
    while (T--) {
        char op;
        int i, j;
        cin >> op >> i >> j;
        if (op == 'M') {
            merge(i, j, 1);
        } else {
            int fi = find(i), fj = find(j);
            if (fi != fj) {
                cout << -1 << '\n';
            } else {
                cout << abs(w[i] - w[j]) - 1 << '\n';
            }
        }
    }
    return 0;
}
```

### [P1525 [NOIP2010 提高组] 关押罪犯](https://www.luogu.com.cn/problem/P1525)

#### 题目大意

给定 $n$ 个节点和 $m$ 条边，求将每个点分配到两个独立的互不相连的图后，两个图中边权的最大值的最小值。

若可以通过一种分配方式使得每个图（森林）中的点互不相连，输出 `0` 。

#### 题目分析

考虑种类并查集。设 `fa[i]` 表示监狱 A ，`fa[i + n]` 表示监狱 B 。

我们不妨对每条边按照边权降序排序，每次对于一条边的两个端点，尝试将其分配到不同的两个集合中。若当前边的两个端点已经在同一集合中，则代表此边为产生冲突的最大边权的最小值。

证明显然。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 10;

int n, m;
struct Relation {
    int a, b, c;

    bool operator > (const Relation x) const {
        return c > x.c;
    }
} relations[N];
int fa[2 * N];

int find(int x) {
    if (fa[x] == x) return x;
    return fa[x] = find(fa[x]);
}

void merge(int u, int v) {
    int fu = find(u), fv = find(v);
    if (fu != fv) {
        fa[fu] = fv;
    }
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        scanf("%d%d%d", &relations[i].a, &relations[i].b, &relations[i].c);
    }
    sort(relations + 1, relations + m + 1, greater<Relation>());
    for (int i = 1; i <= 2 * n; i++) {
        fa[i] = i;
    }
    for (int i = 1; i <= m; i++) {
        if (find(relations[i].a) == find(relations[i].b) || find(relations[i].a + n) == find(relations[i].b + n)) {
            printf("%d\n", relations[i].c);
            return 0;
        }
        merge(relations[i].a, relations[i].b + n);
        merge(relations[i].a + n, relations[i].b);
    }
    printf("0\n");
    return 0;
}
```

### [P1783 海滩防御](https://www.luogu.com.cn/problem/P1783)

#### 题目大意

给定在纵方向上无限延伸的二维平面 $U$ 和平面内 $m$ 个点，其中每个点可以覆盖以 $k$ 为半径的圆形区域。二维平面的最大横坐标为 $n$ 。求一个最小的 $k$ ，使得这 $m$ 个点覆盖范围所形成的区域的横坐标可以连续覆盖 $[0, n]$ 区间。

精度 $\geq 10^{-2}$ 。

#### 题目分析

考虑二分答案。

对于 `check` 函数，我们不妨对每两个防御塔构成的点对 $(t_i, t_j)$ 按照距离进行排序预处理。对于每次 `check` 操作，假设当前答案为 $x$ ，则我们不妨先选择出所有相距距离 $\leq x$ 的防御塔点对，然后根据每个（已连接或未连接的）防御塔距离两个边界 $(0,0)$ 和 $(n, 0)$ 的大小进行判断是否符合。

我们可以使用并查集来维护该操作。首先先将所有距离 $\leq x$ 的防御塔点对都 `merge` 到相应集合中，然后对于每个需要检查距离边界距离的点对 $(i,j)$ ，若 $i$ 和 $j$ 在同一集合中且 $i$ ，$j$ 均符合距离要求，则当前答案 $x$ 满足条件，缩小右端点；反之亦然。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e6 + 10;
constexpr double eps = 1e-4;

int n, m;

class DisjointSet {
private:
    int fa[N];

public:
    DisjointSet() {
        for (int i = 0; i < N; i++) {
            fa[i] = i;
        }
    }

    int find(int x) {
        if (fa[x] == x) return x;
        return fa[x] = find(fa[x]);
    }

    void merge(int u, int v) {
        int fu = find(u), fv = find(v);
        if (fu != fv) {
            fa[fu] = fv;
        }
    }
};

struct Point {
    int x, y;

    Point() {}

    Point(int _x, int _y) {
        x = _x, y = _y;
    }

    double distanceFrom(const Point a) const {
        return sqrt(double(x - a.x) * (x - a.x) + double(y - a.y) * (y - a.y));
    }
};

struct Tower {
    int id;
    Point pos;
} a[N];

struct DiffPair {
    Tower u, v;

    bool operator< (const DiffPair a) const {
        return u.pos.distanceFrom(v.pos) < a.u.pos.distanceFrom(a.v.pos);
    }
};

vector<DiffPair> vt;

bool check(double x) {
    DisjointSet ds;
    for (auto pair : vt) {
        if (pair.u.pos.distanceFrom(pair.v.pos) > 2 * x) {
            break;
        }
        ds.merge(pair.u.id, pair.v.id);
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= m; j++) {
            Tower L = a[i], R = a[j];
            int fL = ds.find(L.id), fR = ds.find(R.id);
            if (L.pos.x <= x && R.pos.x + x >= n && fL == fR) {
                return 1;
            }
        }
    }
    return 0;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        int col, row;
        scanf("%d%d", &col, &row);
        a[i].id = i, a[i].pos.x = col, a[i].pos.y = row;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= m; j++) {
            vt.push_back(DiffPair{a[i], a[j]});
        }
    }
    sort(vt.begin(), vt.end(), less<DiffPair>());
    double l = 0, r = n;
    while (r - l >= eps) {
        double mid = (l + r) / 2;
        if (check(mid)) {
            r = mid;
        } else {
            l = mid;
        }
    }
    printf("%.2lf\n", l);
    return 0;
}
```

### [P2502 [HAOI2006] 旅行](https://www.luogu.com.cn/problem/P2502)

#### 题目大意

给定一张 $n$ 个点 $m$ 条边的带权无向图，求一条从节点 $s$ 到节点 $t$ 的路径，使得路径上最大权值与最小权值的比值最小。输出该比值。

数据保证边权均为正数。

#### 题目分析

乍一看很像 Kruskal 重构树，那不妨运用 Kruskal 的部分思路进行求解。

首先，我们可以通过一个并查集来判断 $s$ 与 $t$ 的连通性，若不连通则直接输出 `IMPOSSIBLE` 结束程序。

接下来考虑求解。我们不妨对边按照权值从小到大排序，按照边权大小枚举最小边和最大边。每次扩展最大边时，我们将该边的两端点使用并查集进行连通，若某一时刻 $s$ 与 $t$ 在当前并查集中连通，则尝试更新答案并结束枚举。

该做法正确性显然：选择了最小边、最大边后，我们无需关注中间边的连接方式和边权，因为它们对于最终答案没有贡献。

需要注意的是，存在 $s$ 与 $t$ 仅需要一条边相连的情况。为此，我们只需要将最大边枚举的左边界移动为当前最小边即可。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e3 + 10;
const int INF = 0x3f3f3f3f;

int n, m, s, t;

class DisjointSet {
private:
    int fa[N];

public:
    DisjointSet() {
        for (int i = 0; i < N; i++) {
            fa[i] = i;
        }
    }

    int find(int x) {
        if (fa[x] == x) return x;
        return fa[x] = find(fa[x]);
    }

    void merge(int u, int v) {
        int fu = find(u), fv = find(v);
        if (fu != fv) {
            fa[fu] = fv;
        }
    }
};

struct Edge {
    int u, v, w;

    bool operator <(const Edge a) const {
        return w < a.w;
    }
};
vector<Edge> edge;

DisjointSet ds;

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; i++) {
        int u, v, w;
        scanf("%d%d%d", &u, &v, &w);
        edge.push_back(Edge{u, v, w});
        ds.merge(u, v);
    }
    scanf("%d%d", &s, &t);
    if (ds.find(s) != ds.find(t)) {
        printf("IMPOSSIBLE\n");
        return 0;
    }
    sort(edge.begin(), edge.end());
    int maxn = INF, minn = INF;
    double ans = INF;
    for (int i = 0; i < edge.size(); i++) {
        DisjointSet edgeDs;
        for (int j = i; j < edge.size(); j++) {
            Edge minEdge = edge[i], currEdge = edge[j];
            edgeDs.merge(currEdge.u, currEdge.v);
            if (edgeDs.find(s) == edgeDs.find(t)) {
                double curr = (double)currEdge.w / (double)minEdge.w;
                if (curr < ans) {
                    ans = curr;
                    maxn = currEdge.w, minn = minEdge.w;
                }
                break;
            }
        }
    }
    if (maxn % minn == 0) {
        printf("%d\n", maxn / minn);
    } else {
        printf("%d/%d\n", maxn / __gcd(maxn, minn), minn / __gcd(maxn, minn));
    }
    return 0;
}
```

### [P2391 白雪皑皑](https://www.luogu.com.cn/problem/P2391)

#### 题目大意

对一个长度为 $n$ 的序列进行 $m$ 次染色操作，第 $i$ 次染色操作将第 $[(i \times p + q) \bmod n] + 1$ 至第 $[(i \times q + p) \bmod n] + 1$ 个数染色为 $i$ （包括端点）。求 $m$ 次操作之后的染色序列。

初始时染色序列均为 $0$ 。

#### 题目分析

一道使用并查集维护**序列信息**的题目。

我们容易发现一个性质：若从第 $m$ 次操作开始倒序枚举，则每个数仅会被染色一次。我们不妨利用这个性质，记录对于每个节点 $i$ ，其需要被染色的下一个节点 $\text{fa}_i$ 。

考虑使用并查集实现。直接暴力枚举 $m$ 次操作，对于每次操作暴力枚举区间 $[l, r]$ 。设当前数为 $i$ ，则我们从并查集中找到其对应的下一个需要染色的点 $\text{nxt}_i$ 进行跳转。若 $i = \text{nxt}_i$ ，则说明当前节点尚未被染色，我们将其染色后将 $\text{fa}_i$ 与 $\text{fa}_{i + 1}$ 进行合并，代表其下一个需要染色的节点为 $\text{fa}_{i+1}$ 所对应的节点。

由于并查集优秀的传递性，我们可以高效的进行暴力枚举。若并查集实现为渐进 $O(1)$ ，则此实现方法时间复杂度约为 $O(m)$ ，接近线性。

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e6 + 10;

int n, m, p, q;

int color[N];

class DisjointSet {
private:
    int fa[N];

public:
    DisjointSet() {
        for (int i = 0; i < N; i++) {
            fa[i] = i;
        }
    }

    int find(int x) {
        if (fa[x] == x) return x;
        return fa[x] = find(fa[x]);
    }

    void merge(int u, int v) {
        int fu = find(u), fv = find(v);
        if (fu != fv) {
            fa[fu] = fv;
        }
    }
} ds;

int main() {
    scanf("%d%d%d%d", &n, &m, &p, &q);
    for (int i = m; i >= 1; i--) {
        int l = (i * p + q) % n + 1,
        r = (i * q + p) % n + 1;
        if (l > r) swap(l, r);
        for (int j = l; j <= r;) {
            int nxt = ds.find(j);
            if (nxt == j) {
                color[j] = i;
                ds.merge(j, j + 1);
            }
            j = nxt;
        }
    }
    for (int i = 1; i <= n; i++) {
        printf("%d\n", color[i]);
    }
    return 0;
}
```

