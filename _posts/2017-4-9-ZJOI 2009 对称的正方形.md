---
title: ZJOI 2009 对称的正方形 Manacher
layout: post
tag: Manacher
category: 省选
intro: Orez很喜欢搜集一些神秘的数据，并经常把它们排成一个矩阵进行研究。最近，Orez又得到了一些数据，并已经把它们排成了一个n行m列的矩阵。通过观察，Orez发现这些数据蕴涵了一个奇特的数，就是矩阵中上下对称且左右对称的正方形子矩阵的个数.Orez自然很想知道这个数是多少，可是矩阵太大，无法去数。只能请你编个程序来计算出这个数。
---

Description
---

Orez很喜欢搜集一些神秘的数据，并经常把它们排成一个矩阵进行研究。最近，Orez又得到了一些数据，并已经把它们排成了一个n行m列的矩阵。通过观察，Orez发现这些数据蕴涵了一个奇特的数，就是矩阵中上下对称且左右对称的正方形子矩阵的个数。Orez自然很想知道这个数是多少，可是矩阵太大，无法去数。只能请你编个程序来计算出这个数。

Input
---

文件的第一行为两个整数$n$和$m$。接下来$n$行每行包含$m$个正整数，表示Orez得到的矩阵。

Output
---

文件中仅包含一个整数$answer$，表示矩阵中有$answer$个上下左右对称的正方形子矩阵。

Solution
---

首先在数之间加上0，矩阵变为$(2n-1)\times(2m-1)$，于是就只要找边长为奇数的正方形了。

不妨设$left[i][j]$为以$(i, j)$为正方形中心时，仅考虑左边方向所能延伸的最长长度$,right[][],up[][],down[][]$定义类似；设$lx[i][j]$为以$(i，j)$为中心的，水平方向上的最长回文串长度的一半，$ly[][]$定义类似；设$f[i][j]$为以$(i，j)$为中心的合法正方形个数。则可得到以下结论（证明略）:

1）$$f[i][j] = \min\{left[i][j], right[i][j], up[i][j], down[i][j]\}$$

2）$$left[i][j] = \max\{ k\mid \min\{ly[i][u], u \in [j-k, j]\} >= j - k\}$$,$right[][],up[][],down[][]$ 求法类似；

3）$i$ 一定时，$j - left[i][j]$随j增加不减，$right[][]，up[][]，down[][]$性质类似；

4）若$i, j$同偶，则$(i，j)$对答案的贡献为$\frac{f[i][j] + 1}2$；若i,j同奇，则其贡献为$\frac{f[i][j]}{2} + 1$；否则没有贡献。

结合以上结论，最小值用ST解决，回文串用马拉车解决即可。

Source
---

<pre><code class="c++">#include &lt;cstdio>
#include &lt;iostream>
#include &lt;cstring>
#include &lt;algorithm>

#define For(i,j,k) for(int i = j;i <= k;i++)
#define Forr(i,j,k) for(int i = j;i >= k;i--)
#define Set(i,j) memset(i, j, sizeof(i))

const int N = 2010;

using namespace std;

int n, m;
int A[N][N], v[N];
int lx[N][N], ly[N][N];
int Min[N][12], Log[N], f[N][N];

int Read(){
    char c = getchar();
    while(c > '9' || c < '0') c = getchar();
    int x = c - '0';
    for(c = getchar();'0' <= c && c <= '9';c = getchar()) x = x * 10 + c - '0';
    return x;
}

void Manacher(int *len, int tot){
    int k = 0;
    For(i,1,tot){
        int p = k + len[k];
        if(p > i && len[2 * k - i] < p - i) len[i] = len[2 * k - i];
        else{
            if(p > i) len[i] = p - i;
            while(i + len[i] < tot && i - len[i] > 1 && v[i + len[i] + 1] == v[i - len[i] - 1]) ++len[i];
            k = i;
        }
    }
}

void RMQ(int x[N][N], int k, int tot){
    Set(Min, 0x3f);
    For(i,1,tot) Min[i][0] = x[i][k];
    For(j,1,11)
        For(i,1,tot - (1 << j) + 1) Min[i][j] = min(Min[i][j-1], Min[i+(1<<(j-1))][j-1]);
}

int Query(int L, int R){
    int x = Log[R - L + 1];
    return min(Min[L][x], Min[R-(1<<x)+1][x]);
}

int main(){
    n = Read(), m = Read();
    For(i,2,N-1) Log[i] = Log[i>>1] + 1;
    For(i,1,n) For(j,1,m) A[i*2-1][j*2-1] = Read();
    n = n * 2 - 1, m = m * 2 - 1;
    For(i,1,n){
        For(j,1,m) v[j] = A[i][j];
        Manacher(lx[i], m);
    }
    For(i,1,m){
        For(j,1,n) v[j] = A[j][i];
        Manacher(ly[i], n);
    }
    Set(f, 0x3f);
    For(i,1,n){
        RMQ(ly, i, m);
        int v = 1;
        For(j,1,m){
            while(v < j && Query(v, j) < j - v) ++v;
            f[i][j] = min(f[i][j], j - v);
        }
        v = m;
        Forr(j,m,1){
            while(v > j && Query(j, v) < v - j) --v;
            f[i][j] = min(f[i][j], v - j);
        }
    }
    For(i,1,m){
        RMQ(lx, i, n);
        int v = 1;
        For(j,1,n){
            while(v < j && Query(v, j) < j - v) ++v;
            f[j][i] = min(f[j][i], j - v);
        }
        v = n;
        Forr(j,n,1){
            while(v > j && Query(j, v) < v - j) --v;
            f[j][i] = min(f[j][i], v - j);
        }
    }
    int Ans = 0;
    For(i,1,n) For(j,1,m) 
        if((i & 1) && (j & 1)) Ans += (f[i][j] >> 1) + 1;
        else if(!(i & 1) && !(j & 1)) Ans += (f[i][j] + 1) >> 1;
    printf("%d\n", Ans);
    return 0;
}
</code></pre>
