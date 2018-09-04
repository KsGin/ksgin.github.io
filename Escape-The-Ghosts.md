---
title: Escape-The-Ghosts
date: 2018-03-09 13:35:15
tags: LeetCode
---

#### 题目地址

[LeetCode#789 Escape The Ghosts](https://leetcode.com/problems/escape-the-ghosts/description/)

#### 题目描述

&emsp;&emsp;You are playing a simplified Pacman game. You start at the point `(0, 0)`, and your destination is` (target[0], target[1])`. There are several ghosts on the map, the i-th ghost starts at` (ghosts[i][0], ghosts[i][1])`.

<!--more-->

&emsp;&emsp;Each turn, you and all ghosts simultaneously *may* move in one of 4 cardinal directions: north, east, west, or south, going from the previous point to a new point 1 unit of distance away.

&emsp;&emsp;You escape if and only if you can reach the target before any ghost reaches you (for any given moves the ghosts may take.)  If you reach any square (including the target) at the same time as a ghost, it doesn't count as an escape.

&emsp;&emsp;Return True if and only if it is possible to escape.

```
Example 1:
Input: 
ghosts = [[1, 0], [0, 3]]
target = [0, 1]
Output: true
Explanation: 
You can directly reach the destination (0, 1) at time 1, while the ghosts located at (1, 0) or (0, 3) have no way to catch up with you.

```

```
Example 2:
Input: 
ghosts = [[1, 0]]
target = [2, 0]
Output: false
Explanation: 
You need to reach the destination (2, 0), but the ghost at (1, 0) lies between you and the destination.
```

```
Example 3:
Input: 
ghosts = [[2, 0]]
target = [1, 0]
Output: false
Explanation: 
The ghost can reach the target at the same time as you.
```

**Note:**

- All points have coordinates with absolute value <= `10000`.
- The number of ghosts will not exceed `100`.

#### 解题思路

&emsp;&emsp;ghosts 可以四处移动或者静止不动，所以要判断能够安全到达目的地，只需要判断所有鬼魂到达目的地的时间比玩家长就是了，即判断所有鬼魂距离目的地逗比玩家远。

#### 解题代码【.CPP】

```c++
class Solution {
    bool canEscape(vector<int> &ghost , vector<int>& target) {
        auto distance1 = abs(target[0]) + abs(target[1]);
        auto distance2 = abs(target[0]-ghost[0]) + abs(target[1] - ghost[1]);
        return distance1 < distance2;
    }

public:
    bool escapeGhosts(vector<vector<int>> &ghosts, vector<int> &target) {
        for (auto &ghost : ghosts) {
            if(!canEscape(ghost , target)) return false;
        }
        return true;
    }
};
```

