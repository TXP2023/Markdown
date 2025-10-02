
# 简述问题

给定一个长度为 $n$ 的序列 $A$, 进行 $Q$ 次询问，每次询问求出 $[l,r]$ 所有子区间的区间和

# 解决方案

对于每个询问, 暴力做法， 对于区间内元素 $A_i$,如果一个子区间包含这个元素，那么其左端点位于 $[l, i]$, 右端点位于 $[i, r]$， 这个元素的总贡献为 $(i-l + 1)  \times  (r-i + 1)  \times  A_i$

那么对于单个询问的答案就是

 $$\Sigma_{i=l}^{r} A_i  \times  (i-l + 1)  \times  (r-i + 1)$$

 

我们定义

$$mulSum(l, r) = \Sigma_{i=1}^{r}A_i \times i - \Sigma_{i=1}^{l}A_i \times i$$

$$powSum(l, r) = \Sigma_{i=1}^{r}A_i \times i^2 - \Sigma_{i=1}^{l}A_i \times i^2$$

$$sum(l, r) = \Sigma_{i=1}^{r}A_i - \Sigma_{i=1}^{l}A_i$$

显而易见的，这些都是前缀和及其变体， 这些都是可以在 $O(n)$ 的复杂度内预处理的

下面推式子

$$\Sigma_{i=l}^{r} A_i  \times  (i-l + 1)  \times  (r-i + 1)$$

$$\Sigma_{i=l}^{r} A_i  \times  (i \times  (r-i + 1)- l \times  (r-i + 1) +  (r-i + 1))$$

$$\Sigma_{i=l}^{r} A_i  \times  (ir-i^2 + i- lr+il - l +  r-i + 1)$$

$$\Sigma_{i=l}^{r} A_i  \times  (ir-i^2 - lr+il - l +  r + 1)$$

$$\Sigma_{i=l}^{r} A_i  \times  (ir-i^2+il) + A_i \times ( - lr - l +  r + 1)$$

$$\Sigma_{i=l}^{r} A_i  \times  (ir-i^2+il) + Sum(l,r) \times ( - lr - l +  r + 1)$$

$$\Sigma_{i=l}^{r} (A_i  \times  ir- A_i  \times  i^2+ A_i  \times il) + Sum(l,r) \times ( - lr - l +  r + 1)$$

$$\Sigma_{i=l}^{r} (A_i  \times  ir+ A_i  \times il) - powSum(l, r) + Sum(l,r) \times ( - lr - l +  r + 1)$$

$$\Sigma_{i=l}^{r} (A_i  \times  i \times (l+r)) - powSum(l, r) + Sum(l,r) \times ( - lr - l +  r + 1)$$

$$ mulSum(l,r) \times (l+r) - powSum(l, r) + Sum(l,r) \times ( - lr - l +  r + 1)$$


那么我们就可以在 $O(1)$ 的复杂度内完成对于单个询问的计算

总解决方案的复杂度为 $O(n)$ 的预处理 和 $O(q)$ 的所有询问

即 $O(n+q)$

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
#define MAXN          (size_t)(3e5+5)

typedef long long int ll;
typedef unsigned long long int ull;

template<typename _T>
inline _T fpow(_T a, _T n, _T mod) {
    _T base = a, ret = 1;
    while (n) {
        if (n & 1) {
            ret = ret * base;
            ret %= mod;
        }
        base = base * base;
        base %= mod;
        n >>= 1;
    }
    return ret % mod;
}
//快读函数声明
template< typename Type >
inline Type fread(Type* p = nullptr);

//快速输出函数
template<typename Type>
inline void fwrite(Type x);

ull arr[MAXN], pow_arr[MAXN], sum[MAXN], mul[MAXN];
ll n, q;

inline ull get_sum(ll l, ll r) {
    return sum[r] - sum[l - 1];
}

inline ull get_mul(ll l, ll r) {
    return mul[r] - mul[l - 1];
}

inline ull get_pow(ll l, ll r) {
    return pow_arr[r] - pow_arr[l - 1];
}

int main() {

#ifdef _FREOPEN
    freopen("input.txt", "r", stdin);
#endif // _FREOPEN

#ifdef _RUN_TIME
    clock_t start = clock();
#endif // _RUN_TIME

    fread(&n), fread(&q);

    for (size_t i = 1; i <= n; i++) {
        fread(&arr[i]);
        mul[i] = arr[i] * i + mul[i - 1];
        pow_arr[i] = arr[i] * i * i + pow_arr[i - 1];
        sum[i] = arr[i] + sum[i - 1];
    }

    while (q--) {
        ll l, r;
        fread(&l), fread(&r);
        ull ans = get_mul(l, r) * (l + r) - get_pow(l, r) + get_sum(l, r) * (-l * r - l + r + 1);
        printf("%llu\n", ans);
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