---
layout: post
title: Monkey ans Tree
intro: 有一天， Monkey得到了一棵有个$N$节点的树。树上的边用一个三元组$(u_i,v_i,l_i)$来表示，意思是$u_i$和$v_i$之间有一条边长为$l_i$的边。于是可爱的Monkey和她的妹妹Little Monkey在树上开心地玩耍。她们要一起玩游戏。因为Monkey特别喜欢跑步，而且跑得特别快，所以她们两人决定在树上跑步。可起点和终点怎么决定呢？Monkey决定从$M$个点对$(a_i,b_i)$，中选出两个点对$(a_i,b_i)$ 和 $(a_j,b_j)$。然
tag: 点分治
category: TUOJ
---

Description
---

有一天， Monkey得到了一棵有个$N$节点的树。树上的边用一个三元组$(u_i,v_i,l_i)$来表示，意思是$u_i$和$v_i$之间有一条边长为$l_i$的边。于是可爱的Monkey和她的妹妹Little Monkey在树上开心地玩耍。她们要一起玩游戏。因为Monkey特别喜欢跑步，而且跑得特别快，所以她们两人决定在树上跑步。可起点和终点怎么决定呢？Monkey决定从$M$个点对$(a_i,b_i)$，中选出两个点对$(a_i,b_i)$ 和 $(a_j,b_j)$。然后Monkey从$a_i$跑到$a_j$，Little Monkey从$b_i$跑到$b_j$。她们姐妹俩希望她们两人跑过的总路程尽量长。但是Monkey还有别的事要做，你能帮帮她吗？

两个人跑过的总路程定义为两人各自的路程之和。单个人的路程定义为起点和终点路径上的边权和。

Input
---

$N\ M$

$u_i\ v_i\ l_i$

$a_i\ b_i$

$n, m \le 10^5, l_i \le 10^9, TL = 4s$

Output
---

输出一个整数表示答案。

Solution
---

有一个部分分是所有$a_i$都相同，这个时候只要考虑$b_i$，显然是一个点分治点经典问题，每次考虑经过分治中心的路径即可。

那么有两条路径时我们同样可以点分治，每次考虑$a_i$到$a_j$的路径经过当前分治重心的情况。

想到这里就不会了。。正解需要先把分治结构（点分树）保存下来，预处理出每一层的分治重心到联通块内每一个点的距离。再进行点分治，对于每一个分治重心$A$分开考虑。对于一个在当前点联通块内的$a_i$，依次考虑它对应的$b_i$在点分树上的祖先$B$，将$dis(A, a_i) + dis(B, b_i)$作为信息在放在$B$上，那么对每个访问到的节点我们只需要找到来自不同子树的两条最大的信息即可，这个只需要记录一下最大和次大值。

由于点分树的深度是$\log$的，总的复杂度就是$O(m \log^2 n + n \log n)$.

需要注意的是次大值的处理以及不要忽略了$a_i = a_j$的情况。

Source
---

```c++
#include <bits/stdc++.h>

#define pb push_back
#define For(i, j, k) for(int i = j; i <= k; i++)

using namespace std;

typedef long long LL;

const int N = 1e5 + 10, K = 25;

int Begin[N], Next[N << 1], to[N << 1], len[N << 1], e;

void add(int u, int v, int w){
	to[++e] = v, Next[e] = Begin[u], Begin[u] = e, len[e] = w;
}

int n, m;
int fa[N][K], bel[N][K], dep[N];
LL dis[N][K];
vector<int> sub[N];

int center, nowsz, minsz;

int DFS_center(int o, int f){
	int sum = 1, mx = 0;
	for(int i = Begin[o]; i; i = Next[i]){
		int u = to[i];
		if(dep[u] || u == f) continue;
		int ch = DFS_center(u, o);
		mx = max(mx, ch);
		sum += ch;
	}
	mx = max(mx, nowsz - sum);
	if(mx < minsz) minsz = mx, center = o;
	return sum;
}

int cnt[N];

void DFS_dis(int o, int rt, int f){
	int k = dep[rt];
	fa[o][k] = rt;
	sub[rt].pb(o);
	cnt[o] = 1;
	for(int i = Begin[o]; i; i = Next[i]){
		int u = to[i];
		if(dep[u] || u == f) continue;
		dis[u][k] = dis[o][k] + len[i];
		bel[u][k] = o == rt ? u : bel[o][k];
		DFS_dis(u, rt, o);
		cnt[o] += cnt[u];
	}
}

void Divide_Conquer(int rt, int sz, int d){	
	nowsz = sz, minsz = sz;
	DFS_center(rt, 0);
	int o = center;
	dep[o] = d;

	DFS_dis(o, o, 0);

	for(int i = Begin[o]; i; i = Next[i]){
		int u = to[i];
		if(!dep[u]) Divide_Conquer(u, cnt[u], d + 1);
	}
}

vector<int> A[N];

LL Max1[N], Max2[N];
int from1[N], from2[N];
int stk[N * K], top;
LL ans;

void check(LL w, int o){
	For(i, 1, dep[o]){
		int v = fa[o][i];
		ans = max(ans, w + dis[o][i] + (bel[o][i] == from1[v] ? Max2[v] : Max1[v]));
	}
}

void insert(LL w, int o){
	For(i, 1, dep[o]){
		int v = fa[o][i];
		LL d = w + dis[o][i];
		if(d > Max1[v]){
			if(from1[v] != bel[o][i]) Max2[v] = Max1[v], from2[v] = from1[v];
			Max1[v] = d, from1[v] = bel[o][i];
		}
		else if(d > Max2[v] && from1[v] != bel[o][i])
			from2[v] = bel[o][i], Max2[v] = d;
		stk[++top] = v;
	}
}

int main(){

	scanf("%d%d", &n, &m);
	For(i, 2, n){
		int u, v, w;
		scanf("%d%d%d", &u, &v, &w);
		add(u, v, w), add(v, u, w);
	}
	Divide_Conquer(1, n, 1);

	For(i, 1, m){
		int a, b;
		scanf("%d%d", &a, &b);
		A[a].pb(b);
	}

	For(i, 1, n){
		int k = dep[i], sz = sub[i].size();
		For(j, 0, sz - 1){
			int l = j, r = j;
			while(r < sz - 1 && bel[sub[i][r + 1]][k] == bel[sub[i][l]][k]) ++r;
			j = r;
			if(j){
				For(u, l, r){
					int o = sub[i][u];
					for(int v : A[o]) check(dis[o][k], v);
				}
				For(u, l, r){
					int o = sub[i][u];
					for(int v : A[o]) insert(dis[o][k], v);
				}
			}
			else For(u, l, r){
				int o = sub[i][u];
				for(int v : A[o]) check(dis[o][k], v), insert(dis[o][k], v);
			}
		}

		while(top){
			int o = stk[top--];
			Max1[o] = Max2[o] = -(1ll << 60);
			from1[o] = from2[o] = 0;
		}

	}
	printf("%lld\n", ans);

	return 0;
}
```
