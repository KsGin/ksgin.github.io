---
title: Target-Sum
date: 2018-01-05 08:34:52
tags: LeetCode
---

[LeetCode#494 Target Sum](https://leetcode.com/problems/target-sum/description/)

&emsp;&emsp;You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols `+` and `-`. For each integer, you should choose one from `+` and `-` as its new symbol.

&emsp;&emsp;Find out how many ways to assign symbols to make sum of integers equal to target S.

<!--more-->

**Example 1:**

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.

```

**Note:**

1. The length of the given array is positive and will not exceed 20.
2. The sum of elements in the given array will not exceed 1000.
3. Your output answer is guaranteed to be fitted in a 32-bit integer.

#### 解题思路

&emsp;&emsp;这道题是需要用数组里的所有数字经过+或者-的运算后得到所要求的数字。用一个DFS递归完所有可能性，满足情况的给结果加一就可以了。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int findTargetSumWays(vector<int> &nums, int S) {
        int res = 0;
        findWays(nums, S, 0, res);
        return res;
    }
    void findWays(vector<int>& atoms, int target, int dep, int &res) {
        if (dep >= atoms.size()) {
            if(target == 0) ++res;
            return;
        }
        findWays(atoms, target - atoms[dep], dep + 1, res);
        findWays(atoms, target + atoms[dep], dep + 1, res);
    }
};
```

