---
title: Swim-in-Rising-Water
date: 2018-03-12 17:45:32
tags: LeetCode
---

#### 题目地址

[LeetCode#778 Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/description/)

#### 题目描述

&emsp;&emsp;On an N x N `grid`, each square `grid[i][j]` represents the elevation at that point `(i,j)`.

&emsp;&emsp;Now rain starts to fall. At time `t`, the depth of the water everywhere is `t`. You can swim from a square to another 4-directionally adjacent square if and only if the elevation of both squares individually are at most `t`. You can swim infinite distance in zero time. Of course, you must stay within the boundaries of the grid during your swim.

&emsp;&emsp;You start at the top left square `(0, 0)`. What is the least time until you can reach the bottom right square `(N-1, N-1)`?

<!--more-->

**Example 1:**

```
Input: [[0,2],[1,3]]
Output: 3
Explanation:
At time 0, you are in grid location (0, 0).
You cannot go anywhere else because 4-directionally adjacent neighbors have a higher elevation than t = 0.

You cannot reach point (1, 1) until time 3.
When the depth of water is 3, we can swim anywhere inside the grid.
```

**Example 2:**

```
Input: [[0,1,2,3,4],[24,23,22,21,5],[12,13,14,15,16],[11,17,18,19,20],[10,9,8,7,6]]
Output: 16
Explanation:
 0  1  2  3  4
24 23 22 21  5
12 13 14 15 16
11 17 18 19 20
10  9  8  7  6

The final route is marked in bold.
We need to wait until time 16 so that (0, 0) and (4, 4) are connected.
```

**Note:**

1. `2 <= N <= 50`.
2. grid[i][j] is a permutation of [0, ..., N*N - 1].

#### 解题思路

&emsp;&emsp;题目意思是给定一个矩阵，里边的每个数字代表着第 N 秒才可以从这里通过，最终要求从左上角到右下角找出一条能最早到达的路径，也就是说一路上经过的数字都尽可能最小。

&emsp;&emsp;这道题思路基本上就是优先队列（Priority Queue）了，将经过的值都 push 进去，然后 pop 队首（因为是优先队列，所以是能到达的最小值），再从 pop 继续扩散。当到达右下角的时候返回。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    struct T {
        int t, x, y;
        T(int a, int b, int c) : t (a), x (b), y (c){}
        bool operator< (const T &d) const {return t > d.t;}
    };

    int swimInWater(vector<vector<int>>& grid) {
        int N = grid.size (), res = 0;
        priority_queue<T> pq;
        pq.push(T(grid[0][0], 0, 0));
        vector<vector<int>> seen(N, vector<int>(N, 0));
        seen[0][0] = 1;
        static int dir[4][2] = {{0, 1}, {0, -1}, {1, 0}, { -1, 0}};

        while (true) {
            auto p = pq.top ();
            pq.pop ();
            res = max(res, p.t);
            if (p.x == N - 1 && p.y == N - 1) return res;
            for (auto& d : dir) {
                int i = p.x + d[0], j = p.y + d[1];
                if (i >= 0 && i < N && j >= 0 && j < N && !seen[i][j]) {
                    seen[i][j] = 1;
                    pq.push (T(grid[i][j], i, j));
                }
            }
        }
    }
};
```

