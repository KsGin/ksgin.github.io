---
title: Longest-Continuous-Increasing-Subsequence
date: 2018-03-24 13:26:16
tags: LeetCode
---

#### 题目地址

[LeetCode#674 Longest Continuous Increasing Subsequence](https://leetcode.com/problems/longest-continuous-increasing-subsequence/description/)

#### 题目描述

&emsp;&emsp;Given an unsorted array of integers, find the length of longest `continuous` increasing subsequence (subarray).

<!--more-->

**Example 1:**

```
Input: [1,3,5,4,7]
Output: 3
Explanation: The longest continuous increasing subsequence is [1,3,5], its length is 3. 
Even though [1,3,5,7] is also an increasing subsequence, it's not a continuous one where 5 and 7 are separated by 4. 
```

**Example 2:**

```
Input: [2,2,2,2,2]
Output: 1
Explanation: The longest continuous increasing subsequence is [2], its length is 1. 
```

**Note:** Length of the array will not exceed 10,000.

#### 解题思路

&emsp;&emsp;直接遍历，当连续增长中断的时候更新最大值。

#### 解题代码

```c++
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if (nums.empty()) return 0;
        int res = 1 , cot = 1;
        for (int i = 1; i < nums.size(); ++i) {
            if(nums[i] > nums[i-1]) cot++;
            else {
                res = max(cot , res);
                cot = 1;
            }
        }
        res = max(cot , res);
        return res;
    }
};
```

