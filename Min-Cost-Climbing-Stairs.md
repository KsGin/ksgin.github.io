---
title: Min-Cost-Climbing-Stairs
date: 2018-01-21 13:35:29
tags: LeetCode
---

#### 题目地址

[LeetCode#746 Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/description/)

#### 题目描述

On a staircase, the `i`-th step has some non-negative cost `cost[i]` assigned (0 indexed).

Once you pay the cost, you can either climb one or two steps. You need to find minimum cost to reach the top of the floor, and you can either start from the step with index 0, or the step with index 1.

<!--more-->

**Example 1:**

```
Input: cost = [10, 15, 20]
Output: 15
Explanation: Cheapest is start on cost[1], pay that cost and go to the top.
```

**Example 2:**

```
Input: cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
Output: 6
Explanation: Cheapest is start on cost[0], and only step on 1s, skipping cost[3].
```

**Note:**

1. `cost` will have a length in the range `[2, 1000]`.
2. Every `cost[i]` will be an integer in the range `[0, 999]`.

#### 解题思路

&emsp;&emsp;这种全局最优解的题要么递归要么动态规划，这道题递归超时，动态规划可以直接解。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int size = static_cast<int>(cost.size());
        vector<int> dp(size,0);
        dp[0] = cost[0];
        dp[1] = cost[1];
        for (int i = 2; i < size; ++i) {
            dp[i] = cost[i] + min(dp[i-1] , dp[i-2]);
        }
        return min(dp[size-1],dp[size-2]);
    }
};
```

