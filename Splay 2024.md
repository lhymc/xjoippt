# Splay

## 核心

Splay 的核心：Rotate 和 Splay 操作，每次操作节点旋转至根，其实就是**旋转**。

维护的信息：节点权值，左右孩子，出现次数，子树大小，父亲节点，根节点



## 旋转

目的：让 Splay 尽可能**平衡**。

​	对于任意节点 $p$ 而言，如果以 $p$ 为根的子树发生了**旋转**，那么这个子树的根肯定不为 $p$。

​	根据根节点旋转后在子树中的位置，可以发现旋转分为左旋和右旋，容易发现，它们其实是对称的可以归纳在一起写。

![](https://oi-wiki.org/ds/images/splay-rotate.svg)

```cpp
void rotate(int x)
{
    int y = fa[x], z = fa[y], chk = get(x);
    ch[y][chk] = ch[x][chk ^ 1];
    if (ch[x][chk ^ 1]) fa[ch[x][chk ^ 1]] = y;
    ch[x][chk ^ 1] = y;
    fa[y] = x; fa[x] = z;
    if (z) ch[z][y == ch[z][1]] = x;
    maintain(y); maintain(x);
}
```

或者发现很容易写错，于是可以死记硬背。

```cpp
int u(f[p]), x(f[u]), q(ChildSide(p));
Child[u][q] = Child[p][q ^ 1];
f[f[f[Child[u][q]] = Child[p][q ^ 1] = u] = p] = x;
if (x) Child[x][r(x) == u] = p;
Update(u); Update(p);
```



## Splay

​	如果要把一个点旋转到整棵树的根节点，怎么办，要旋转好多次，每次旋转是怎么样的呢？

​	旋转分为三类，如果记录方向，分为六种。

1. $\text{Zig(Zag)}$

   旋转一次

   ![](https://oi-wiki.org/ds/images/splay-zig.svg)

2. $\text{Zig-Zig(Zag-Zag)}$

   旋转两次，但是两次方向一样

​	![](https://oi-wiki.org/ds/images/splay-zig-zig.svg)

3. $\text{Zag-Zig(Zig-Zag)}$

   旋转两次，但是两次方向不同

   ![](https://oi-wiki.org/ds/images/splay-zig-zag.svg)

   这几种情况确实是存在的，Splay 函数的意义在于旋转至根节点。

   ```c++
   inline void Splay(int p)
   {
   	for (int q(p); q = f[p]; Rotate(p))
   		if (f[q]) Rotate(p ^ r(f[p]) ^ q ^ r(f[q]) ? p : q);
   	Root = p;
   }
   ```
   
   注意：$\text{Zag}$ 是左旋，$\text{Zig}$ 是右旋。



## 插入

1. 树是空的
2. 树上已经有了与该点值相同的节点
3. 树上没有与该点值相同的节点

```C++
inline void Insert(int p)
{
	if (!Root) { t = Root = 1; s[1] = c[1] = 1; v[1] = p; return; }
	int u(Root), x(0);
	while (true) {
		if (v[u] == p) { ++c[u]; ++s[u]; ChangeSize(x); Splay(u); return; }
		x = u; u = cu(v[u] < p);
		if (!u) {
			v[++t] = p; c[t] = s[t] = 1; f[t] = x;
			Child[x][v[x] < p] = t; ChangeSize(x);
			Splay(t); return;
}   }   }
```



## 排名

​	从根开始搜索，搜索到当前节点 $p$。

1. 当前节点值为 $p$。
2. 当前节点值小于 $p$。
3. 当前节点值大于 $p$。

```c++
inline int Rank(int p)
{
	int u(Root), x(0), i(0);
	while (++i)
		if (p < v[u]) u = l(u);
		else {
			x += s[l(u)];
			if (p == v[u]) Sret(x + 1);
			x += c[u]; u = r(u);
}		}
```

​	如果要查询排名为 $k$ 的数，容易发现，这跟排名本质上是互逆的。

```C++
inline int Pth(int p)
{
	int u(Root);
	while (true)
		if (l(u) && p <= s[l(u)]) u = l(u);
		else {
			p -= c[u] + s[l(u)];
			if (p < 1) Sret(v[u]);
			u = r(u);	
}		}
```



## 前驱 / 后继

​	前驱是当前节点左子树中最大的节点权值，后继是当前节点右子树中最小的节点权值，同排名，搜索即可。

```C++
int pre()
{
    int cur = ch[rt][0];
    if (!cur) return cur;
    while (ch[cur][1]) cur = ch[cur][1];
    splay(cur);
    return cur;
}
```

```c++
int nxt()
{
    int cur = ch[rt][1];
    if (!cur) return cur;
    while (ch[cur][0]) cur = ch[cur][0];
    splay(cur);
    return cur;
}
```

​	也有简便一些合在一起的写法。

```C++
inline int PreSuc(bool p)
{
	int u(p ? r(Root) : l(Root)); 
	while (p ? l(u) : r(u)) u = p ? l(u) : r(u); Sret(u);
}
```

注意：在某些情境中，查询的数不一定已经存在于平衡树中，所以要先把它插进去，查询完后再删除。



## 删除

1. 该点是叶子节点
2. 该点只有左子树
3. 该点只有右子树
4. 该点两棵子树都有，实质上，这就是合并两棵子树

```c++
inline void Delete(int p)
{
	Rank(p); int u(Root), x;
	if (c[u] > 1) { --c[u]; --s[u]; return; }
	if (!l(u) && !r(u)) { Clear(u); Root = t = 0; return; }
	if (!l(u)) { f[Root = r(u)] = 0; Clear(u); return; }
	if (!r(u)) { f[Root = l(u)] = 0; Clear(u); return; }
	f[r(u)] = x = PreSuc(false); r(x) = r(u); Clear(u);
}
```



## 合并 / 区间删除

​	合并两棵子树 $x,y$，并不是归并排序，而是要求 $\max x<\min y$，只要把 $x$ 最大值旋转到根，将 $y$ 接上即可。

​	同理可以进行区间删除，进行两次旋转之后，将一段区间清除即可。



## 时间复杂度

​	均摊达到 $O(n\log n)$，可以感性理解，具体证明需要使用势能分析，可以见 [OI-WIKI](https://oi-wiki.org/ds/splay/#splay-%E6%93%8D%E4%BD%9C%E7%9A%84%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6)。



## 代码

```c++
#include <stdio.h>
const int _(100005);
int Root, t, n, opt, y, f[_], Child[_][2], v[_], c[_], s[_];
#define l(p) Child[p][0]
#define r(p) Child[p][1]
#define cp(z) Child[p][z]
#define cu(z) Child[u][z]
#define Sret(z) { Splay(u); return (z); }
inline void ChangeSize(int p) { s[p] = s[l(p)] + s[r(p)] + c[p]; }
inline bool ChildSide(int p) { return p == r(f[p]); }
inline void Clear(int p) { l(p) = r(p) = f[p] = v[p] = c[p] = s[p] = 0; }
inline void Rotate(int p)
{
	int u(f[p]), x(f[u]), q(ChildSide(p)); cu(q) = cp(q ^ 1);
	f[f[f[cu(q)] = cp(q ^ 1) = u] = p] = x;
	if (x) Child[x][r(x) == u] = p;
	ChangeSize(u); ChangeSize(p);
}
inline void Splay(int p)
{
	for (int q(p); q = f[p]; Rotate(p))
		if (f[q]) Rotate(p ^ r(f[p]) ^ q ^ r(f[q]) ? p : q);
	Root = p;
}
inline void Insert(int p)
{
	if (!Root) { t = Root = 1; s[1] = c[1] = 1; v[1] = p; return; }
	int u(Root), x(0);
	while (true) {
		if (v[u] == p) { ++c[u]; ++s[u]; ChangeSize(x); Splay(u); return; }
		x = u; u = cu(v[u] < p);
		if (!u) {
			v[++t] = p; c[t] = s[t] = 1; f[t] = x;
			Child[x][v[x] < p] = t; ChangeSize(x);
			Splay(t); return;
}   }   }
inline int Rank(int p)
{
	int u(Root), x(0), i(0);
	while (++i)
		if (p < v[u]) u = l(u);
		else {
			x += s[l(u)];
			if (p == v[u]) Sret(x + 1);
			x += c[u]; u = r(u);
}		}
inline int Pth(int p)
{
	int u(Root);
	while (true)
		if (l(u) && p <= s[l(u)]) u = l(u);
		else {
			p -= c[u] + s[l(u)];
			if (p < 1) Sret(v[u]);
			u = r(u);	
}		}
inline int PreSuc(bool p)
{
	int u(p ? r(Root) : l(Root)); 
	while (p ? l(u) : r(u)) u = p ? l(u) : r(u); Sret(u);
}
inline void Delete(int p)
{
	Rank(p); int u(Root), x;
	if (c[u] > 1) { --c[u]; --s[u]; return; }
	if (!l(u) && !r(u)) { Clear(u); Root = t = 0; return; }
	if (!l(u)) { f[Root = r(u)] = 0; Clear(u); return; }
	if (!r(u)) { f[Root = l(u)] = 0; Clear(u); return; }
	f[r(u)] = x = PreSuc(false); r(x) = r(u); Clear(u);
}
int main()
{
	for (scanf("%d", &n); n; --n) {
		scanf("%d%d", &opt, &y);
		if (opt < 2) Insert(y);
		else if (opt < 3) Delete(y);
		else if (opt < 5) { printf("%d\n", opt < 4 ? Rank(y) : Pth(y));}
		else { Insert(y); printf("%d\n", v[PreSuc(opt - 5)]); Delete(y); }
	} return 0;
}
```



---

## 维护信息

​	其实 Splay 可以当线段树用，因为我们发现它支持懒标记，只需要适当的添加 $\text{pushup}$ 和 $\text{pushdown}$ 即可。

​	此外，它还可以维护一些更厉害的，例如**区间翻转**。

​	区间翻转可以看作对一段区间加了一个翻转标记。在 Splay 上可以用 Splay 函数把这段区间旋转成一棵子树，我们只要对这整棵子树加翻转标记即可。两个翻转标记会**抵消**。

​	一般而言，平衡树都是以权值为排序，但是它还可以以编号排序。其实这个按编号排序还是比较无聊的，因为它几乎没有“排序”，所以它其实可以用别的数据结构代替。但是别的数据结构几乎不支持区间反转。于是得到一个结论：区间翻转与平衡树强相关。而且平衡树可以有线段树所有的功能。



## 其它操作

· 定向 Splay：将某个点定向旋转到另一个点的位置

· $O(n)$ 建树：用于“不用排序”的 Splay，类似与线段树的建树。



## 例题

[LOJ 104 普通平衡树](https://loj.ac/p/104)

[LOJ 105 普通平衡树](https://loj.ac/p/105)

[P2710 数列](https://www.luogu.com.cn/problem/P2710)



## 其它应用

​	如果你学有余力：可以学习 LCT，可持久化平衡树，树套树和动态图完全连通性

[LOJ 186 动态树](https://loj.ac/p/186)

[P3835 【模板】可持久化平衡树](https://www.luogu.com.cn/problem/P3835)

[P5055 【模板】可持久化文艺平衡树](https://www.luogu.com.cn/problem/P5055)

[LOJ 106 二逼平衡树](https://loj.ac/p/106)

[LOJ 122 动态图连通性](https://loj.ac/p/122)



## $\text{STL}$

可以代替平衡树的 $$\text{STL}$$：`multiset`。

支持：普通平衡树的所有操作。

**Rope** : 可持久化平衡树

```c++
#include <ext/rope>
using namespace __gnu_cxx;
```

常用函数：
```c++
rope<int> s;
int v, w, t;
auto p(lower_bound(s.begin(), s.end(), v));
s.insert(p, w);
s.erase(p, 1);
s[t] = v;
```

​	但是 `rope` 本质是块状链表，时空间复杂度均为 $O(n\sqrt n)$。其优点在于：**区间删除**，**$O(1)$ 拷贝**。

​	当然还有 `pb_ds`，功能强大，可以自行学习。
