
## 概述

栈是一种 LIFO(Last In First Out)表，意思是后进入的元素先出来，你可以将它理解为一个木桶。

一张图（图源 [OI-wiki](https://oi-wiki.org/ds/stack/)）：

![stack.svg](https://oi-wiki.org/ds/images/stack.svg)

栈是一种比较通用的线性数据结构，在递归，表达式求值，括号配对等地方有很多应用。

比如你的 dfs，本质上是用了一个系统栈在模拟，（所以会有手工栈这种优化方式）。

栈一般支持几个操作：

+ `push(x)`：将元素 $x$ 入栈，复杂度 $O(1)$。
+ `pop()`：将栈顶元素 $top$ 出栈，复杂度 $O(1)$。
+ `gettop()`：询问栈顶元素 $top$ 的值，复杂度 $O(1)$。

可以使用数组模拟：

```cpp
int st[si], tp = 0; // tp 模拟栈顶指针，初始为空。

void push(int x) { st[++tp] = x; }
void pop(){ if(tp) --tp; }
int gettop(){ return st[tp]; }
void clear(){ tp = 0; } // 清空栈，这是数组模拟的好处，直接指向 0 就行了。
```

也可以直接使用 STLstack，包含在 `#include <stack>` 中。

```cpp
std::stack<int>s;

int x;

s.push(x); // 压入。
cout << s.top() << endl;
s.pop(); // 弹栈

int sz = int(s.size()) // 求其大小
bool ept = s.empty() // 是否为空

```

## 应用

### 单调栈

见：[单调栈](../monotonous-stack)。

### 表达式求值

见：[表达式求值](../../misc/expression.md)。
