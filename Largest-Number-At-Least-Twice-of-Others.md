---
title: Largest-Number-At-Least-Twice-of-Others
date: 2018-01-20 23:16:38
tags: LeetCode
---

#### 题目地址

[LeetCode#747 Largest Number At Least Twice of Others](https://leetcode.com/problems/largest-number-at-least-twice-of-others/description/)

#### 题目描述

In a given integer array `nums`, there is always exactly one largest element.

Find whether the largest element in the array is at least twice as much as every other number in the array.

If it is, return the **index** of the largest element, otherwise return -1.

<!--more-->

**Example 1:**

```
Input: nums = [3, 6, 1, 0]
Output: 1
Explanation: 6 is the largest integer, and for every other number in the array x,
6 is more than twice as big as x.  The index of value 6 is 1, so we return 1.
```

**Example 2:**

```
Input: nums = [1, 2, 3, 4]
Output: -1
Explanation: 4 isn't at least as big as twice the value of 3, so we return -1.
```

**Note:**

1. `nums` will have a length in the range `[1, 50]`.
2. Every `nums[i]` will be an integer in the range `[0, 99]`.

#### 解题思路

&emsp;&emsp;题目要求是找出数组里满足是最大数且至少是其他数字的两倍的数字，返回其索引，如果没有就返回-1。我们通过一次遍历找出数组的最大值和次大值，然后判断最大值是不是次大值得两倍或者更多，是的话返回最大值索引，不是返回-1。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int dominantIndex(vector<int>& nums) {
        int ldx = 0 , sdx = -1;
        for (int i = 1; i < nums.size(); ++i) {
            if(nums[i] > nums[ldx]){
                sdx = ldx;
                ldx = i;
            } else {
                if(sdx == -1 || nums[i] > nums[sdx]) {
                    sdx = i;
                }
            }
        }
        return nums[ldx] >= nums[sdx] * 2 ? ldx : -1;
    }
};
```

