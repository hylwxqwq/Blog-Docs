## 泛化

这一类问题都是询问某个区间 $[L,R]$ 当中**满足某个条件的数**的个数。

并且 $L,R$ 的**范围一般很大**

常见的限制有：

1. 各位数字和不能是某个整数的倍数
2. 某一个数字不能出现
3. 数字必须按照升序排列
4. 不能出现某两个连续的数字

## 实现

套路无非两步：

+ 先把 $F(L),F(R)$ 算出来，答案就是 $F(R)-F(L-1)$。

  其中 $F(X)$ 表示 $[1,X]$ 当中满足条件的数的个数。

+ 考虑每一位怎么填，是否有上界限制或者前导零限制。

  上界限制是这样的：比如你现在要求 $F(n)$ 。

  并且 $n=12345678$

  我们从最高位，也就是 $8$ 所处的位置开始处理。

  那么这一位就有两种情况：

  + 这一位填 $8$ ，也就是说下一位只能填 $0 \sim 7$。
  + 这一位填 $0\sim 7$ 的某一个数，也就是说下一位，包括之后的所有位想怎么填怎么填（$0 \sim 9$）

  这个就直接开一个变量记录一下，来确定当前的决策集合是什么（可以选择的数字有哪些）。

  然后前导零限制也是直接记录一下即可（如果前导零标记为 True 则表示从开始搜索一直到现在都填的是 $0$）。

这个就是在 DP，设 $dp(x, y, 0/1, 0/1)$ 表示满足什么条件在什么情况下的答案，不过为了实现方便，通常会写记忆化搜索。

```pascal
function dfs(当前位数 x, 当前状态 y, 前导零限制 st, 上界限制 limit):
	if 到达边界 and 符合要求 then 
  	返回边界的合法答案
  if 到达边界 and 不符合要求 then
    返回边界的不合法答案          # 这个一般不会有，一般枚举填数的时候如果没有限制就会有（只要会访问到边界不合法情况就要加上）。 

  if 当前的状态已经记忆化过 then
  	返回记录的答案

  var result = 0 	     # 记录答案
	var up = 9 	         # 当前位填数的上限
	if 有上界限制 then
		up=当前位在 n 当中的数字       # n 是要求的 F(n) 的自变量
	
	for 枚举当前位的填数值 from 0 to up :
		if 当前位填的数不符合限制 then
			continue
		if 有前导零限制 and 当前填写的是 0 then 
       	result += dfs(x - 1, 下一个状态, true, 是否触碰上界限制)
    else then
        result += dfs(x - 1, 下一个状态, false, 是否触碰上界限制)

  记录当前状态的答案 f[x][y] 	= result
  返回答案 result


function solve(要求的F的自变量 n)
	存储每一位数字的 vector 清空
    while n!=0 then
		vector <== n%base 			# base表示是哪一个进制
		n/=base
  清空状态数组
	返回对应状态的答案（调用 dfs）
```

有几个需要注意的点：

+ 当前状态 $y$ 需要由题目的限制决定，比如 上一个填写的/当前的数位总和/已经填了多少位 这种限制。
+ 边界一般是 $x=0$，有的时候需要判断当前状态是否合法（比如是否是某个数的倍数之类的），而且边界的答案要看情况而定。
+ 第 15 行的“限制”，这个需要看题目要求，比如有没有哪一个数不能出现，哪两个数不能连续出现（这个需要在开始的时候记录上一位）的要求。
+ 前导零限制不是所有题都会有，如果有前导零限制，在递归下一个状态的时候记得让下一个状态强制合法（因为在前导零限制下，实际上这一位是没有填数字的，而不是填了 0）。

## 例题

来一道经典题（包含所有要素）：

> [SCOI2009] Windy数：不含前导零且相邻两个数字之差至少为 $2$ 的正整数被称为 Windy 数。
>
> 问你 $L \sim R$ 当中有多少个 Windy 数。$1\le L \le R \le 2\times 10^9$ 
>
> （我最开始有个错，区间是默认 $L\not=R$ 的，所以如果要相等就不能写“区间”，某次模拟赛的题也因为这个差点被喷（

套板子就行了，根本不需要过多的思考。

注意这个不含前导零的意思是，`0012`, `012`, `12` 算作同一个方案。

直接上代码，有几个小的注意点已经写在里面了：

??? note "Code"
  ```cpp
  // author : black_trees

  #include <cstdio>
  #include <vector>
  #include <cstring>
  #include <iostream>
  #include <algorithm>

  #define endl '\n'

  using namespace std;
  using i64 = long long;

  const int si = 15 + 10;
  int L, R;
  std::vector<int>dight;
  int f[si][si][2][2];

  int dfs(int x, int y, int st, int limit) {
    if(!x) return 1;
    if(~f[x][y][st][limit]) return f[x][y][st][limit];
    int ret = 0, up = 9;
    if(limit) up = dight[x - 1];
    for(int i = 0; i <= up; ++i) {
      if(abs(i - y) < 2) continue; // 不合法
      if(st && i == 0)
        ret += dfs(x - 1, 11, 1, limit && i == up); // 强制合法
      else ret += dfs(x - 1, i, 0, limit && i == up);
    }
    f[x][y][st][limit] = ret;
    return ret;
  }
  int solve(int n) {
      dight.clear();
      while(n) dight.push_back(n % 10), n /= 10;
      memset(f, -1, sizeof f);            // 注意清空
      return dfs(dight.size(), 11, 1, 1); // 强制合法
  }

  int main() {  

    cin.tie(0) -> sync_with_stdio(false);
    cin.exceptions(cin.failbit | cin.badbit);

    cin >> L >> R;
    cout << solve(R) - solve(L - 1) << endl;

    return 0;
  }
  ```

还有几个需要特殊注意的点：

!!! Warning
    根据状态来开记忆化的数组的大小，不要开小了，也要小心负下标！。
    
    清空记忆化数组的时机也需要把握。
    	
    如果不管怎么询问需要的状态是一样的，那么可以在 `main()` 的最开头写 `memset`
    
    如果每一个询问底下会不一样，那么要在 `solve` 或者当前询问底下写 `memset`（这种常见一点）。
