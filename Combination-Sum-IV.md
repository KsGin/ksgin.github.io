---
title: Combination-Sum-IV
date: 2018-03-15 11:50:04
tags: 
	- LeetCode
---

#### 题目地址

[LeetCode#377 Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/description/)

#### 题目描述

&emsp;&emsp;Given an integer array with all positive numbers and no duplicates, find the number of possible combinations that add up to a positive integer target.

<!--more-->

**Example:**

```
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.

Therefore the output is 7.
```

**Follow up:**
&emsp;&emsp;What if negative numbers are allowed in the given array?
&emsp;&emsp;How does it change the problem?
&emsp;&emsp;What limitation we need to add to the question to allow negative numbers?

#### 解题思路

&emsp;&emsp;看到题后下意识的用了递归，然后就超时了。。。。

&emsp;&emsp;这道题的解法应该是 DP ，递归公式则为 

&emsp;&emsp;dp[i] = {sum(dp[i-x1] , dp[i-x2] , dp[i-x3] …...) | x in nums && nums[x] < i}

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int combinationSum4(vector<int> &nums, int target) {
        vector<int> dp(target+1,0);
        dp[0] = 1;
        for (int i = 1; i <= target; ++i) {
            for (int j = 0; j < nums.size(); ++j) {
                if (i >= nums[j]) dp[i] += dp[i - nums[j]];
            }
        }

        return dp[target];
    }
};
```

