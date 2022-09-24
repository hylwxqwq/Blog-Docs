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

+ 考虑每一位怎么填，是否有上界限制（利用树的思想）

  上界限制是这样的：比如你现在要求 $F(n)$ 。

  并且 $n=12345678$

  我们从最高位，也就是 $8$ 所处的位置开始处理。

  那么这一位就有两种情况：

  + 这一位填 $8$ ，也就是说下一位只能填 $0 \sim 7$。
  + 这一位填 $0\sim 7$ 的某一个数，也就是说下一位，包括之后的所有位想怎么填怎么填（$0 \sim 9$）

  第一种情况就是有上界限制的。

当然，还有一些其他的细节，就直接看一份伪代码吧

??? node "Code"
  ```pascal
  function dfs(当前位数 x,当前状态 y,前导零限制 st,上界限制 limit):
  	if 到达边界 then 
  	返回边界的答案
  
  if 没有上界限制 and 当前的状态已经记忆化过 then
  	返回记录的答案
  
  variable result=0 	# 记录答案
  
  	variable up=9 	    # 当前位填数的上限
  	if 有上界限制 then
  		up=当前位在 n 当中的数字 # n 是要求的 F(n) 的自变量
  	
  	for 枚举当前位的填数值 from 0 to up :
  		if 当前位填的数不符合限制 then
  			continue
  		if 有前导零限制 and 当前填写的是 0 then
         	result+=dfs(x-1,下一个状态,true,是否触碰上界限制)
        else then
          	result+=dfs(x-1,下一个状态,false,是否触碰上界限制)
     	
    if 没有前导零限制 and 没有上界限制 then
     	记录当前状态的答案 f[x][y]
     	
    返回答案 result
  
  
  
  function solve(要求的F的自变量 n)
  	存储每一位数字的 vector 清空
      while n!=0 then
  		vector <== n%base 			# base表示是哪一个进制
  		n/=base
  	返回对应状态的答案（调用 dfs）
  ```

从上到下解释一下：

+ Line1 ：当前状态 $y$ 需要由题目的限制决定，比如 上一个填写的/当前的数位总和/已经填了多少位 这种限制

+ Line2 ：边界一般是 $x=0$，有的时候需要判断当前状态是否合法（比如是否是某个数的倍数之类的）

+ Line3 ：边界的答案要看情况而定，有的时候是 $0,1$ 有的时候是当前填出来的这个数的平方。

+ Line5 ：这里一般使用两个 bool 变量 $st,limit$，直接判断就可以了。

+ Line12 ：我记录每一位的数字一般用 vector 直接 pb，所以如果当前位是 $x$，那么这里 $up=dight_{x-1}$ 即可。

+ Line15 ：这个需要看题目限制，比如有没有哪一个数不能出现，哪两个数不能连续出现（这个需要在开始的时候记录上一位）

+ Line18 ：使用 `limit && i==up` 这个语句即可。

+ Line34 ：这个也是要看题目限制而定，不过有几个通用的细节需要注意。

  1. 最开始，如果是从高位开始搜索，需要把 limit 设置为 true 。
  
     因为最高位本就有限制（后面的大小不能随便填了）。	
  
  
  2. 从高位开始搜索，那么开始的时候要设置前导零限制为 true。
  
     因为前导零本来就会存在于最高位。
  
  3. 如果 $y$ 记录的是上一步填的是什么，一般会填 $0$，当然，有个题填的是 $-2$ ，这个技巧到时候再说。
  

## 例题

来一道经典题（包含所有要素）：

> [SCOI2009] Windy数：不含前导零且相邻两个数字之差至少为 $2$ 的正整数被称为 Windy 数。
>
> 问你 $L \sim R$ 当中有多少个 Windy 数。$1\le L \le R \le 2\times 10^9$ 
>
> （我最开始有个错，区间是默认 $L\not=R$ 的，所以如果要相等就不能写“区间”，某次模拟赛的题也因为这个差点被喷（

套板子就行了，根本不需要过多的思考。

直接上代码：

??? note "Code"
  ```cpp
  /*
   * @Author: black_trees <black_trees@foxmail.com>
   * @Date: 2022-02-12 16:32:04
   * @Last Modified by: black_trees <black_trees@foxmail.com>
   * @Last Modified time: 2022-02-12 17:08:53
   */
  
  /*
   * READ THIS BEFORE YOU WRITE YOUR CODE
   * ====================================
   * @ Read the description carefully!
   * @ Pay attention to the data domain.
   * @ Keep calm.
   * @ Write down your thought on your draft.
   * @ Keep your mind clear.
   * @ Stabilize the coding speed.
   * ====================================
   * READ THIS AGAIN BEFORE YOU START !!!
   */
  #include<bits/stdc++.h>
  using namespace std;
  
  constexpr int si=15;
  int L,R;
  std::vector<int>dight;
  int f[si][si];
  
  inline int dfs(int x,int y,int st,int limit){
  	if(!x) return 1;
  	if(!limit && f[x][y]!=-1) return f[x][y];
  	int res=0,up=9; 
  	if(limit) up=dight[x-1];
  	for(register int i=0;i<=up;++i){
  		if(abs(i-y)<2) continue;
  		if(st && i==0) res+=dfs(x-1,-2,1,limit && i==up);
  		else res+=dfs(x-1,i,0,limit && i==up);
  	} if(!st && !limit) f[x][y]=res;
  	return res;
  }
  
  inline int solve(int n){
  	dight.clear();
  	while(n) dight.push_back(n%10),n/=10;
  	return dfs(dight.size(),-2,1,1); // 填 -2 是为了让开始的时候不管选什么数都可以满足差大于2的条件。
  }
  
  int main(){
  	memset(f,-1,sizeof f),scanf("%d%d",&L,&R);
  	return printf("%d\n",solve(R)-solve(L-1)),0;
  }
  ```

还有几个需要特殊注意的点：

!!! Warning
    根据状态来开记忆化的数组的大小，不要开小了。
    
    清空记忆化数组的时机也需要把握。
    	
    如果不管怎么询问需要的状态是一样的，那么可以在 `main()` 的最开头写 `memset`
    
    如果每一个询问底下会不一样，那么要在 `solve` 或者当前询问底下写 `memset`。
