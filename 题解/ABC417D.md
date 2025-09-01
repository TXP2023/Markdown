# 题目

高桥即将收到 $N$ 份礼物。

高桥有一个名为“情绪值”的非负整数，每收到一份礼物，他的情绪值就会发生变化。每份礼物都有价值 $P$、情绪值上升度 $A$、情绪值下降度 $B$ 三个参数，高桥的情绪值会根据这些参数按以下规则变化：

- 当收到的礼物的价值 $P$ 大于或等于现在的情绪值时，高桥对此礼物感到开心，情绪值增加 $A$。
- 当收到的礼物的价值 $P$ 小于当前情绪值时，高桥对此礼物感到失望，情绪值减少 $B$。但是如果高桥原本的情绪值小于 $B$，他的情绪值会变为 $0$。

第 $i\,(1 \le i \le N)$ 份收到的礼物的价值为 $P_i$、情绪值上升度为 $A_i$、情绪值下降度为 $B_i$。

有 $Q$ 个询问，请回答所有询问。在第 $i\,(1 \le i \le Q)$ 个询问中，给定一个非负整数 $X_i$，请回答以下问题：

> 当高桥最初的情绪值为 $X_i$ 时，求他收到所有 $N$ 份礼物后的情绪值。

#### 数据范围

对于 $100\%$ 的数据保证：

- $1 \le N \le 10000$
- $1 \le P_i \le 500\,(1 \le i \le N)$
- $1 \le A_i \le 500\,(1 \le i \le N)$
- $1 \le B_i \le 500\,(1 \le i \le N)$
- $1 \le Q \le 5 \times 10^5$
- $0 \le X_i \le 10^9\,(1 \le i \le Q)$
- 输入的所有数均为整数。

# Slove

## 暴力做法  

对于每个询问，暴力计算每次收到礼物。 复杂度 $O(N*Q)$, 无法通过 

## 正解

寻找突破口

> $1 \le P_i, A_i, B_i  \le 500\,(1 \le i \le N)$

这三个值太小了， 不正常，从此入手

那么可以发现， 高桥所能达到的最大的情绪值实在当前礼物的 $P = 500, A = 500$ 且当前高桥的情绪值为 $500$ 的情况下， 那么在接收这个礼物后，高桥的情绪值就为 $500+500=1000$

### 分类讨论

那么对于每个 $X$, 存在三种情况

- 这个 $X$ 极大， 使得对于每个礼物 $i$, 在接受这个礼物时的情绪值都严格大于这个礼物的价值，那么 $X$ 就会一直减少。 
- 这个 $X$ 小于最大可能情绪值$1000$ 
- 这个 $X$ 大于最大可能情绪值$1000$, 但是在接收礼物的过程中会小于$1000$

区分这三种情况，可以预处理出 $B$ 的前缀和

### 优化模拟过程

时间开销大的主要原因是很多情况被计算了多次

尝试记录对于接受第一个礼物时，情绪值在 $[0, 1000]$ 的情况， 

容易发现， 第一个礼物的情况可以由第二个礼物的情况转移而来， 第二个礼物可以由第三个礼物转移而来

那么明显的 $dp$ 了， 由于每个礼物的状态都要从下一个礼物的状态转移而来， 那么倒序计算

记从第 $i$ 个礼物，情绪值为 $j$ 的情况下，到接受完最后一个礼物的的情绪值为 $dp[i][j]$

那么存在两种情况

- 这个情绪值礼物的价值， 那么价值会减少， 即从 $dp[i][std::max(j - gift[i].b, 0)]$ 转移 

- 这个情绪值小于礼物的价值， 那么价值会增加， 即从 $dp[i][j + gift[i].a]$ 转移 

```cpp
if (j > gifts[i].p) {
    dp[i][j] = dp[i + 1][std::max(j - gifts[i].b, (ll)0)];
}
else {
    dp[i][j] = dp[i + 1][j + gifts[i].a];
}
```


那么虚拟出第 $n + 1$ 个礼物, 初始化 $dp[n+1][j] = j$

现在解决了两种情况 


- 1）这个 $X$ 极大， 使得对于每个礼物 $i$, 在接受这个礼物时的情绪值都严格大于这个礼物的价值，那么 $X$ 就会一直减少。 
- 2） 这个 $X$ 小于最大可能情绪值$1000$ 

还有一种情况没解决

- 3）这个 $X$ 大于最大可能情绪值$1000$, 但是在接收礼物的过程中会小于$1000$

可以先使用之前计算的 $B$ 的前缀和， 二分查找第一个接受完小于$1000$的礼物，然后转化为情况二解决

```cpp
 ll val = fread<ll>();
 if (val - preSum[n] > MAX_VAL) {
     printf("%lld\n", val - preSum[n]);
     continue;
 }
 if (val <= MAX_VAL) {
     printf("%lld\n", dp[1][val]);
     continue;
 }
 ll pos = std::lower_bound(preSum + 1, preSum + 1 + n, val - MAX_VAL) - preSum;
 printf("%lld\n", dp[pos + 1][val - preSum[pos]]);
```

完整代码

```cpp
//+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
//
//      By txp2024 www.luogu.com.cn  TXP2023 www.github.com
// 
//+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-

#pragma once
#include <vector>
#include <stdio.h>
#include <string.h>
#include <algorithm>
#include <numeric>
#include <ctype.h>
#include <cstdarg>
#include <climits>
#include <time.h>
#include <iostream>
#include <stdint.h>

#define _FREAD        true
#define MAX_INF       1e18
#define MAX_NUM_SIZE  35
#define MAXN          (size_t)(1e4+5)
#define MAX_VAL       1000

typedef long long int ll;
typedef unsigned long long int ull;

//快读函数声明
template< typename Type >
inline Type fread(Type* p = nullptr);

//快速输出函数
template<typename Type>
inline void fwrite(Type x);

struct Gift {
    ll p, a, b;
};

Gift gifts[MAXN];
ll preSum[MAXN];
ll dp[MAXN][MAX_VAL + 5];
ll n, q;

int main() {

#ifdef _FREOPEN
    freopen("input.txt", "r", stdin);
#endif // _FREOPEN

#ifdef _RUN_TIME
    clock_t start = clock();
#endif // _RUN_TIME

    fread(&n);

    for (size_t i = 1; i <= n; i++) {
        fread(&gifts[i].p), fread(&gifts[i].a), fread(&gifts[i].b);
        preSum[i] = preSum[i - 1] + gifts[i].b;
    }

    for (size_t i = 0; i <= MAX_VAL; i++) {
        dp[n + 1][i] = i;
    }

    for (ll i = n; i >= 1; --i) {
        for (ll j = 0; j <= MAX_VAL; j++) {
            if (j > gifts[i].p) {
                dp[i][j] = dp[i + 1][std::max(j - gifts[i].b, (ll)0)];
            }
            else {
                dp[i][j] = dp[i + 1][j + gifts[i].a];
            }
        }
    }

    fread(&q);

    while (q--) {
        ll val = fread<ll>();
        if (val - preSum[n] > MAX_VAL) {
            printf("%lld\n", val - preSum[n]);
            continue;
        }
        if (val <= MAX_VAL) {
            printf("%lld\n", dp[1][val]);
            continue;
        }
        ll pos = std::lower_bound(preSum + 1, preSum + 1 + n, val - MAX_VAL) - preSum;
        printf("%lld\n", dp[pos + 1][val - preSum[pos]]);
    }


#ifdef _RUN_TIME
    printf("The running duration is not less than %ld ms\n", clock() - start);
#endif // _RUN_TIME
    return 0;
}

template< typename Type >
inline Type fread(Type* p) {
#if _FREAD
    Type ret = 0, sgn = 0, ch = getchar();
    while (!isdigit(ch)) {
        sgn |= ch == '-', ch = getchar();
    }
    while (isdigit(ch)) ret = ret * 10 + ch - '0', ch = getchar();
    if (p != nullptr) {
        *p = Type(sgn ? -ret : ret);
    }
    return sgn ? -ret : ret;
#else
    if (p == nullptr) {
        Type temp;
        p = &temp;
    }
    scanf("%lld", p);
    return *p;


#endif // _FREAD
}


template<typename Type>
inline void fwrite(Type x) {
    int sta[MAX_NUM_SIZE];
    int top = 0;
    do {
        sta[top++] = x % 10, x /= 10;
    } while (x);
    while (top) {
        putchar(sta[--top] + '0');
    }  // 48 是 '0'
    return;
}



/**
 *              ,----------------,              ,---------,
 *         ,-----------------------,          ,"        ,"|
 *       ,"                      ,"|        ,"        ,"  |
 *      +-----------------------+  |      ,"        ,"    |
 *      |  .-----------------.  |  |     +---------+      |
 *      |  |                 |  |  |     | -==----'|      |
 *      |  |                 |  |  |     |         |      |
 *      |  |  C:\>rp++       |  |  |     |`---=    |      |
 *      |  |                 |  |  |     |==== ooo |      ;
 *      |  |                 |  |  |     |(((( [33]|    ,"
 *      |  `-----------------'  | /      |((((     |  ,"
 *      +-----------------------+/       |         |,"
 *         /_)______________(_/          +---------+
 *    _______________________________
 *   /  oooooooooooooooo  .o.  oooo /,   /-----------
 *  / ==ooooooooooooooo==.o.  ooo= //   /\--{)B     ,"
 * /_==__==========__==_ooo__ooo=_/'   /___________,"
 *
 */
```