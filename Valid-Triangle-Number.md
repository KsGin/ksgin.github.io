---
title: Valid-Triangle-Number
date: 2018-05-25 10:39:40
tags: LeetCode
---

#### 题目地址

[LeetCode#611 Valid Triangle Number](https://leetcode.com/problems/valid-triangle-number/description/)

#### 题目描述

&emsp;&emsp;Given an array consists of non-negative integers, your task is to count the number of triplets chosen from the array that can make triangles if we take them as side lengths of a triangle.

<!--more-->

**Example 1:**

```
Input: [2,2,3,4]
Output: 3
Explanation:
Valid combinations are: 
2,3,4 (using the first 2)
2,3,4 (using the second 2)
2,2,3
```

**Note:**

1. The length of the given array won't exceed 1000.
2. The integers in the given array are in the range of [0, 1000].

#### 解题思路

&emsp;&emsp;emmmm判断三条边能否构成三角形的计算应该不需要解释了，我们无脑的使用三层循环就可以 AC ，不过时间复杂度比较高。

#### 解题代码

```c++
class Solution {
public:
    int triangleNumber(vector<int> &nums) {
        int res = 0;
        for (int i = 0; i < nums.size(); ++i) {
            int n1 = nums[i];
            for (int j = i + 1; j < nums.size(); ++j) {
                int n2 = nums[j];
                for (int k = j + 1; k < nums.size(); ++k) {
                    int n3 = nums[k];
                    res += !(n1 + n2 <= n3 || n2 + n3 <= n1 || n1 + n3 <= n2);
                }
            }
        }
        return res;
    }
};
```

