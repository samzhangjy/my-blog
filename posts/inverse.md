---
title: "浅谈乘法逆元"
desc: "乘法逆元可用于快速求解模意义下的大数除法。"
cover: /assets/posts/inverse.jpg
date: 2024.10.05
tags:
  - 算法
  - C++
  - OI
---

乘法逆元在 OI 中有非常重要的用途，通常用来快速求解模意义下大数除法问题。本文将简单讨论有关乘法逆元的求解方法和题目。

## 线性同余方程

在了解乘法逆元概念之前，我们需要先了解一个前置概念：线性同余方程。

形如 $ax \equiv b \pmod n$ 的方程称为**线性同余方程**。其中 $a,b,n$ 为给定整数，$x$ 为未知数。

### 同余

所谓同余，即两个数对同一个数取模有相同的余数。例如 $a \equiv b \pmod p$ 就意味着 $a$ 、$b$ 除以 $p$ 的余数相同。形式化地， $a - b = km$ 为 $a \equiv b \pmod m$ 的充要条件，其中 $k \in \Z$ 。

具象化地，若 $a \equiv b \pmod m$ ，则 $a$ 、$b$ 在$\bmod m$ 意义下是等价的。

同余有如下性质：

- 自反性：$a \equiv a \pmod m$ ；
- 对称性：$a \equiv b \pmod m \Leftrightarrow b \equiv a \pmod m$ ；
- 传递性：$a \equiv b \pmod m ,\ b \equiv c \pmod m \Rightarrow a \equiv c \pmod m$ ；
- 相加：$a \equiv b \pmod m, \ c \equiv d \pmod m \Rightarrow a \pm c \equiv b \pm d \pmod m$ ；
- 相乘：$a \equiv b \pmod m, \ c \equiv d \pmod m \Rightarrow ac \equiv bd \pmod m$ ；
- 幂运算：$a \equiv b \pmod m \Rightarrow a^n \equiv b^n \pmod m$ ；
- ……

我们考虑求解上述线性同余方程。我们可以发现，上述同余方程等价于如下线性不定方程：
$$
ax + nk = b
$$
其中 $x$ 、$k$ 为未知数，当且仅当 $\gcd(a,n) \mid b$ 时上述方程有整数解。

### 扩展欧几里得算法

我们知道，在基本欧几里得算法中，我们通过辗转相除法来求解 $\gcd(a,n)$ 。具体地，基本欧几里得算法有递归式 $\gcd(a,n) = \gcd(n, a \bmod n)$ ，直到 $a \bmod n = 0$ 。

扩展欧几里得算法的扩展在于，它可以在计算 $\gcd(a,n)$ 的基础上求得一组对于方程 $ax + nk = \gcd(a,n)$ 的整数特解。我们称上述方程 $ax + nk = \gcd(a,n)$ 为**贝祖等式**。

扩展欧几里得算法使用了递归的思想求解。我们不妨设已知等式：
$$
nx_1 + (a \bmod n)k_1 = \gcd(n, a \bmod n)
$$
容易发现，$a \bmod n = a - \lfloor\frac{a}{n} \rfloor \times n$ 。则上述等式化为：
$$
nx_1 + (a - \lfloor \frac{a}{n} \rfloor \times n)k_1 = \gcd(n, a\bmod n)
$$
整理得：
$$
ak_1 + n(x_1 - \lfloor \frac{a}{n} \rfloor \times k_1) = \gcd(a,n)
$$
则我们可以求得新的解 $x = k_1$ ，$y = x_1 - \lfloor \frac{a}{b} \rfloor \times k_1$ 。

#### 具体过程

设要计算 $\gcd(a,n)$ 和贝祖等式 $ax + nk = \gcd(a,n)$ 的特解 $x_0$ ，$k_0$ 。

首先，若 $n = 0$ ，则按照基本欧几里得算法，有 $\gcd(a,n) = a$ ，且 $x_0 = 1$ ，$k_0 = 0$ 。递归终止。

我们考虑计算。按照上述推导，我们可以发现 $\gcd(n, a \bmod n) = nx_1 + (a \bmod n)k_1$ ，且当前 $x_0 = y_1$ ，$k_0 = x_1 - \lfloor \frac{a}{n} \rfloor \times k_1$ 。

#### 代码实现

```cpp
int ex_gcd(int a, int b, int& x, int& y) {
    if (b == 0) {
      x = 1;
      y = 0;
      return a;
    }
    int d = ex_gcd(b, a % b, x, y);
    int temp = x;
    x = y;
    y = temp - a / b * y;
    return d;
}
```

### 扩展欧几里得算法求解线性同余方程

我们继续考虑求解线性同余方程 $ax \equiv b \pmod n$ 的转化形式：
$$
ax + nk = b \tag{1-1}
$$

我们已知可以求解出方程
$$
ax_0 + nk_0 = \gcd(a,n) \tag{1-2}
$$
的一组特解。考虑对 $\text{(1-2)}$ 式进行转化。等式两边同时乘以 $\frac{b}{\gcd(a,n)}$ ，得：
$$
a\frac{b}{\gcd(a,n)}x_0 + n\frac{b}{\gcd(a,n)}k_0 = b
$$
则原方程有特解 $x = \frac{b}{\gcd(a,n)}x_0$ ，$k = \frac{b}{\gcd(a,n)}k_0$ 。

考虑求得通解。显然，若 $\gcd(a,n) = 1$ ，且方程 $ax + nk = b$ 有特解 $X_0$ ，$K_0$ ，则我们可以求出该方程的任意解：
$$
\left\{
\begin{aligned}
&x = X_0 + nt,\\
&k = K_0 - at
\end{aligned}
\right.
$$
其中 $t \in \Z$ 。如果要求最小正整数解，则 $x = X_0 \bmod t$ ，其中 $t = \frac{n}{\gcd(a,n)}$ 。若 $n$ 为质数，则 $x = X_0 \bmod n$ 。

#### 代码实现

```cpp
int ex_gcd(int a, int b, int& x, int& y) {
    if (b == 0) {
      x = 1;
      y = 0;
      return a;
    }
    int d = ex_gcd(b, a % b, x, y);
    int temp = x;
    x = y;
    y = temp - a / b * y;
    return d;
}

bool liEu(int a, int b, int c, int& x, int& y) {
    int d = ex_gcd(a, b, x, y);
    if (c % d != 0) return 0;
    int k = c / d;
    x *= k;
    y *= k;
    return 1;
}
```

## 乘法逆元

指模意义下的乘法逆元。

### 定义

如果有 $x$ 满足线性同余方程 $ax \equiv 1 \pmod b$ ，则 $x$ 被称为 $a \bmod b$ 的逆元，记作 $x = a^{-1}$ 。

在实数意义下，$a$ 的逆元即为 $a$ 的倒数 $\frac{1}{a}$ 。在模意义下，$x$ 的逆元 $x^{-1}$ 在模意义下的运算类似于 $\frac{1}{x}$ 。

### 求解

#### 扩展欧几里得求逆元

显然我们可以选择求解上述线性同余方程。直接套用扩展欧几里得算法即可。

```cpp
void exgcd(int a, int b, int& x, int& y) {
  if (b == 0) {
    x = 1, y = 0;
    return;
  }
  exgcd(b, a % b, y, x);
  y -= a / b * x;
}
```

#### 线性求连续序列的逆元

如果要选择求解 $1,2,\dots,n$ 中每个数关于某个模数的逆元，扩展欧几里得算法就会出现很多冗杂的计算，导致时间复杂度偏高。

考虑线性递推求解。我们显然可以注意到一个重要前提：
$$
1^{-1} \equiv 1 \pmod p
$$
证明显然：$1 \times 1 \equiv 1 \pmod p$ 对于任意 $p \in \Z$ 恒成立，$1$ 在$\bmod p$ 意义下的逆元恒为 $1$ 。

假设我们现在要计算 $i$ 的逆元 $i^{-1}$ ，即求出 $i^{-1} \times i \equiv 1 \pmod p$ 的解 $i^{-1}$ 。

令 $k = \lfloor\frac{p}{i} \rfloor$ ，$j = p \bmod i$ 。则 $p = ki + j$ 。即 $ki + j \equiv 0 \pmod p$ 。

两边同时乘以 $i^{-1} \times j^{-1}$ ，有：
$$
kj^{-1} + i^{-1} \equiv 0 \pmod p
$$
移项，有
$$
i^{-1} \equiv -kj^{-1} \pmod p
$$
代入 $k = \lfloor\frac{p}{i}\rfloor$ ， $j = p \bmod i$ ，有
$$
i^{-1} \equiv -\lfloor\frac{p}{i}\rfloor \times (p \bmod i)^{-1} \pmod p
$$
不难发现 $p \bmod i < i$ ，即计算 $i^{-1}$ 时已经算出 $(p \bmod i)^{-1}$ ，可以递推求解。

形式化地，我们有：
$$
i^{-1} = \left \{
\begin{aligned}
&1,&& \text{if}\quad i = 1,\\
&-\lfloor\frac{p}{i}\rfloor \times (p \bmod i)^{-1}, &&\text{otherwise.}\\
\end{aligned}
\right. \pmod p
$$
具体实现如下：

```cpp
inv[1] = 1;
for (int i = 2; i <= n; ++i) {
    inv[i] = (long long)(p - p / i) * inv[p % i] % p;
}
```

注意我们使用 $p - \lfloor\frac{p}{i}\rfloor$ 来防止出现负数。

#### 线性求任意 $n$ 个数的逆元

考虑前缀积。我们计算出 $n$ 个数 $a_1,a_2,\dots,a_n$ 的前缀积 $s_i$ ，然后使用扩展欧几里得算法计算 $s_n$ 的逆元 $s_n^{-1}$ 。

我们不妨抽象理解为实数意义下的逆元，即 $s_n^{-1} = \frac{1}{s_n} \pmod p$ 。

又 $s_n = s_1 \times s_2 \times \dots \times s_n$ ，则 $s_n^{-1} = \frac{1}{s_1 \times s_2 \times \dots \times s_n} \pmod p$ 。将 $s_n^{-1}$ 乘上 $s_n$ ，我们即可得到 $s_{n - 1}^{-1}$ 。如此递推可求得所有 $s_i^{-1}$ 。

对于 $a_i^{-1}$ ，我们显然有：
$$
a_i^{-1} \equiv s_{i - 1} \times s_{i}^{-1} \pmod p
$$

```cpp
s[0] = 1;
for (int i = 1; i <= n; ++i) s[i] = s[i - 1] * a[i] % p;
sv[n] = exgcd(s[n], p);
// sv_i 即为 s_i 的逆元
for (int i = n; i >= 1; --i) sv[i - 1] = sv[i] * a[i] % p;
for (int i = 1; i <= n; ++i) inv[i] = sv[i] * s[i - 1] % p;
```

总时间复杂度为 $O(n + \log p)$ 。

## 题目

### [P1082 [NOIP2012 提高组] 同余方程](https://www.luogu.com.cn/problem/P1082)

模板题，套用扩展欧几里得算法即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e6 + 10;

int a, n;

int exgcd(int a, int n, int& x, int& k) {
    if (n == 0) {
        x = 1, k = 0;
        return a;
    }
    int t = exgcd(n, a % n, x, k);
    int xx = x;
    x = k, k = xx - a / n * k;
    return t;
}

int main() {
    scanf("%d%d", &a, &n);
    int x, k;
    int t = exgcd(a, n, x, k);
    t = n / t;
    printf("%d\n", (x % t + t) % t);
    return 0;
}
```

### [P5431 【模板】模意义下的乘法逆元 2](https://www.luogu.com.cn/problem/P5431)

模板题，使用线性乘法逆元模板带入求解即可。注意快速输入输出和 `long long` 。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 5e6 + 10;

long long n, p, K;
long long a[N], s[N], sv[N];

long long read() {
	long long x=0,flag=1;char c;
	while ((c=getchar())<'0' || c>'9') if(c=='-') flag=-1;
	while (c>='0' && c<='9') x=(x<<3)+(x<<1)+(c^48),c=getchar();
	return x*flag;
}

int exgcd(int a, int n, int& x, int& k) {
    if (n == 0) {
        x = 1, k = 0;
        return a;
    }
    int t = exgcd(n, a % n, x, k);
    int xx = x;
    x = k, k = xx - a / n * k;
    return t;
}

int main() {
    s[0] = 1;
    n = read(); p = read(); K = read();
    for (int i = 1; i <= n; i++) {
        a[i] = read();
        s[i] = s[i - 1] * (a[i] % p) % p;
    }
    int x, k;
    int b = exgcd(s[n], p, x, k);
    b = p / b;
    sv[n] = (x % b + b) % b;
    for (int i = n; i >= 1; i--) {
        sv[i - 1] = sv[i] * a[i] % p;
    }
    long long ans = 0, t = 1, inv;
    for (int i = 1; i <= n; i++) {
        t = t * K % p;
        inv = sv[i] * s[i - 1] % p;
        ans = (ans + inv * t % p) % p;
    }
    printf("%lld\n", ans);
    return 0;
}
```

### [P2054 [AHOI2005] 洗牌](https://www.luogu.com.cn/problem/P2054)

#### 题目大意

给定一个序列 $a_i$ ，其中 $1 \leq i \leq n$ 且 $n$ 为偶数。对于每次操作，将序列分为 $a_1 \sim a_{\frac{n}{2}}$ 和 $a_{\frac{n}{2} + 1} \sim a_n$ 两部分。对于前半部分的元素 $a_i$ ，其在新序列中的位置为 $2i$ ；对于后半部分的元素 $a_j$ ，其在新序列中的位置为 $2j - (n + 1)$ 。

现在给定 $n$ 和操作次数 $m$ ，求经过 $m$ 次操作后位于第 $L$ 个位置上的元素在初始序列中的位置 $x$ 。

其中 $1 \leq n \leq 10^{10}$ ，$0 \leq m \leq 10^{10}$ 且 $n$ 为偶数。

#### 分析

对于序列中任意元素 $a_i$ ，考虑其在新序列中的位置。显然其在新序列中的位置为 $2i \bmod (n + 1)$ 。

我们容易发现，经过 $m$ 次变换后，其最终位置为 $i \cdot 2^m \bmod (n + 1)$ 。则易得如下同余方程：
$$
x \cdot 2^m \equiv L \pmod{n + 1}
$$
则有
$$
x \equiv \frac{L}{2^m} \pmod{n + 1}
$$
考虑乘法逆元。设 $2$ 在$\bmod{n + 1}$ 意义下的乘法逆元为 $k$ ，则有
$$
x \equiv k^m \cdot L \pmod{n + 1}
$$
得解。

对于 $2$ 在模意义下的乘法逆元 $k$ ，我们显然可以通过扩展欧几里得算法在 $O(\log n)$ 时间内求解。

注意到 $n \leq 10^{10}$ ，需要使用快速幂求解 $k^m$ 。这里介绍一下快速乘：一个对于模意义下大数乘法的快速计算方法，原理类似快速幂。具体操作是维护一个被乘数 $a$ ，将其按二进制位依次乘到乘数 $b$ 中，计算时逐步取模。

详细参见 <https://leungll.site/2021/05/24/ksm-permalink/> 和 [https://oi-wiki.org/math/binary-exponentiation](https://oi-wiki.org/math/binary-exponentiation/#%E6%A8%A1%E6%84%8F%E4%B9%89%E4%B8%8B%E5%A4%A7%E6%95%B4%E6%95%B0%E4%B9%98%E6%B3%95) ，存在在 `long long` 范围内的 $O(1)$ 快速解法。下面的程序使用朴素的 $O(\log_2 m)$ 的普通快速乘。

#### 代码

注意 `long long` 和取模。数据范围较大，也需要注意常数优化。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e6 + 10;

long long n, m, L;

long long exgcd(long long a, long long n, long long& x, long long& k) {
    if (n == 0) {
        x = 1, k = 0;
        return a;
    }
    long long t = exgcd(n, a % n, x, k);
    long long xx = x;
    x = k, k = xx - a / n * k;
    return t;
}

long long qmul(long long a, long long b, long long mod) {
    long long ans = 0;
    while (b) {
        if (b & 1) {
            ans = (ans + a) % mod;
        }
        a = (a + a) % mod;
        b >>= 1;
    }
    return ans;
}

long long qpow(long long a, long long b, long long mod) {
    long long ans = 1, base = a;
    while (b) {
        if (b & 1) {
            ans = qmul(ans, base, mod);
        }
        base = qmul(base, base, mod);
        b >>= 1;
    }
    return ans;
}

int main() {
    scanf("%lld%lld%lld", &n, &m, &L);
    long long x, k;
    long long t = exgcd(2, n + 1, x, k);
    t = (n + 1) / t;
    long long inv2 = (x % t + t) % t;
    printf("%lld\n", qmul(L, qpow(inv2, m, n + 1), n + 1));
    return 0;
}
```

