---
title: Single-Number-II
date: 2018-03-19 11:37:54
tags: LeetCode
---

#### 题目地址

[LeetCode#137 Single Number II](https://leetcode.com/problems/single-number-ii/description/)

#### 题目描述

&emsp;&emsp;Given an array of integers, every element appears *three* times except for one, which appears exactly once. Find that single one.

<!--more-->

#### 解题思路

&emsp;&emsp;将数组排序，之后遍历查找就可以了。直接看代码

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        sort(nums.begin() , nums.end());
        bool isDup = false;
        for (int i = 1; i < nums.size(); ++i) {
            if (nums[i] != nums[i-1]){
                if (!isDup) return nums[i-1];
                isDup = false;
            } else {
                isDup = true;
            }
        }
        return nums[nums.size()-1];
    }
};
```

