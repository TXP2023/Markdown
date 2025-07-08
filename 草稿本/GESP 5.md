给定一个有序递增序列a，长度为n，均为正整数

设: $ g = gcd(a_2 - a_1, a_3 - a_2, a_4 - a_3, ... ,a_n - a_{n - 1}) $

对于任意一个正整数k

求证:

存在：

$gcd(a_1 + k, a_2 + k, a_3+k, ..., a_n + k) = gcd(g, a_1 + k)$

证：

设 $a = a_1 + k, b = a_2 + k, c = a_3 + k$

$gcd(a, b, c)$

$=gcd(gcd(a, b), c)$

$=gcd(gcd(a, b - a), c)$

$=gcd(a, b - a, c)$

$=gcd(a, gcd(b - a, c))$

$=gcd(a, gcd(b - a, c - (b - a)))$

$=gcd(a, gcd(b - a, c - b + a))$

$=gcd(a, b - a, c - b + a)$

