
## 概述

队列是一种 FIFO(First In First Out)表，意思是先进入的元素先出来，你可以将它理解为一个管子，从后塞进去，从前面出来。

一张图（图源 [OI-wiki](https://oi-wiki.org/ds/queue/)）：

![queue.svg](https://oi-wiki.org/ds/images/queue.svg)

BFS 和 拓扑排序，SPFA 这种基于”广度优先“的算法，全部都要用到队列来”按层扩展“。

队列一般支持这几种操作：

+ `push(x)`：在队尾加入元素 $x$。
+ `pop()`：弹出队头。
+ `getfront()`：求队头元素的值。
+ `getback()`：求队尾元素的值。

如果拿数组模拟的话，只需要拿两个指针记录头尾即可：

```cpp
int q[si], ql = 1, qr = 0;（这种写法，在单调队列里面会提到为啥）

void push(int x) { q[++qr] = x;}
void pop(int x) { ql++; }
int getfront() { return q[ql]; }
int getback() { return q[qr]; } 
 
int clear() { ql = 1, qr = 0; }
```

你发现这种方法是一直在后移的，久了之后队列的实际大小会缩水。

所以我们可以考虑把 $0$ 和 $\text{SIZE} - 1$ 连起来（如果从 $1$ 连到 $\text{SIZE}$，运算会变麻烦），串成一个环，此时 $ql,qr$ 的初值也要相应改变。

这玩意儿就是循环队列，此时的 `++x` 运算被重新定义为了 $x = (x + 1) \mod \text{SIZE}$

或者使用 STLqueue:

```cpp
std::queue<int>q;

q.push(1), q.push(3); // 插入
cout << q.front() << " " << q.back() << endl;
q.pop(), q.pop(); // 弹出

int sz = int(q.size()) // 求其大小
bool ept = q.empty() // 是否为空

```

## 应用

### 双端队列

就是两边都能出能进的队列。

用 STLdeque 就行了，也可手写（因为 deque 有时常数太大。）

- 元素访问
  - `q.front()` 返回队首元素
  - `q.back()` 返回队尾元素
- 修改
  - `q.push_back()` 在队尾插入元素
  - `q.pop_back()` 弹出队尾元素
  - `q.push_front()` 在队首插入元素
  - `q.pop_front()` 弹出队首元素
  - `q.insert()` 在指定位置前插入元素（传入迭代器和元素）
  - `q.erase()` 删除指定位置的元素（传入迭代器）
- 容量
  - `q.empty()` 队列是否为空
  - `q.size()` 返回队列中元素的数量
 
### 单调队列

见 [单调队列](../monotonous-queue)