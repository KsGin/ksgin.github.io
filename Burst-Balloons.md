---
title: Burst-Balloons
date: 2018-01-31 21:22:23
tags: LeetCode
---

#### 题目地址

[LeetCode#312 Burst Balloons](https://leetcode.com/problems/burst-balloons/description/)

#### 题目描述

Given `n` balloons, indexed from `0` to `n-1`. Each balloon is painted with a number on it represented by array `nums`. You are asked to burst all the balloons. If the you burst balloon `i` you will get `nums[left] * nums[i] * nums[right]` coins. Here `left` and `right`are adjacent indices of `i`. After the burst, the `left` and `right` then becomes adjacent.

<!--more-->

Find the maximum coins you can collect by bursting the balloons wisely.

**Note:** 
(1) You may imagine `nums[-1] = nums[n] = 1`. They are not real therefore you can not burst them.
(2) 0 ≤ `n` ≤ 500, 0 ≤ `nums[i]` ≤ 100

**Example:**

Given `[3, 1, 5, 8]`

Return `167`

```
    nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
   coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167
```

#### 解题思路

&emsp;&emsp;题目是打气球，每打一个气球的得分是这个气球左右两边的数字以及这个气球的数字之积，如果左右两边没有数字就用1代替，最终需要得到可以得到的最大值。

&emsp;&emsp;**求全局极值问题，首先考虑递归或者动态规划！**

&emsp;&emsp;这道题使用动态规划，dp[i]\[j]存放的是nums[i]到nums[j]内可以打爆所有气球获得的最大值，最后我们返回dp[0]\[n]就可以了。

&emsp;&emsp;递推式`dp[i][j] = max(dp[i][j], nums[i - 1]*nums[k]*nums[j + 1] + dp[i][k - 1] + dp[k + 1][j])                 ( i ≤ k ≤ j )`

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        nums.insert(nums.begin(), 1);
        nums.push_back(1);
        vector<vector<int> > dp(nums.size(), vector<int>(nums.size() , 0));
        for (int len = 1; len <= n; ++len) {
            for (int left = 1; left <= n - len + 1; ++left) {
                int right = left + len - 1;
                for (int k = left; k <= right; ++k) {
                    dp[left][right] = max(dp[left][right], nums[left - 1] * nums[k] * nums[right + 1] + dp[left][k - 1] + dp[k + 1][right]);
                }
            }
        }
        return dp[1][n];
    }
};
```



