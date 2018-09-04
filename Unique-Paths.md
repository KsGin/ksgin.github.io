---
title: Unique-Paths
date: 2018-03-18 08:56:00
tags: LeetCode
---

#### 题目地址

[LeetCode#62 Unique Paths](https://leetcode.com/problems/unique-paths/description/)

#### 题目描述

&emsp;&emsp;A robot is located at the top-left corner of a *m* x *n* grid (marked 'Start' in the diagram below).

&emsp;&emsp;The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

<!--more-->

&emsp;&emsp;How many possible unique paths are there?

![img](https://leetcode.com/static/images/problemset/robot_maze.png)

&emsp;&emsp;Above is a 3 x 7 grid. How many possible unique paths are there?

**&emsp;&emsp;Note:** *m* and *n* will be at most 100.

#### 解题思路

&emsp;&emsp;一看到这题就想到无脑递归，但是最后看了看，觉得很有可能会 TEL ，所以放弃选择了 DP 。我们可以简单推出 Start （DP\[0]\[0]） 到 Finish 的路径个数等于他的下边方格（DP\[1][0]）和右边方格（DP\[0][1]）之和。继续往下推也是这个道理，所以最后有递推公式：

`dp[i][j] = dp[i+1][j] + dp[i][j+1];`

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m , vector<int>(n , 0));
        dp[m-1][n-1] = 1;
        for (int i = m-1; i >= 0; --i) {
            for (int j = n-1; j >= 0; --j) {
                if (i+1 < m) dp[i][j] += dp[i+1][j];
                if (j+1 < n) dp[i][j] += dp[i][j+1];
            }
        }
        return dp[0][0];
    }
};
```

