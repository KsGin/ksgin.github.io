---
title: Peak-Index-in-a-Mountain-Array
date: 2018-06-25 19:52:13
tags: LeetCode
---

#### 题目地址

[LeetCode#852 Peak Index in a Mountain Array](https://leetcode.com/problems/peak-index-in-a-mountain-array/description/)

#### 题目描述

&emsp;&emsp;Let's call an array `A` a *mountain* if the following properties hold:

- `A.length >= 3`
- There exists some `0 < i < A.length - 1` such that `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]`

&emsp;&emsp;Given an array that is definitely a mountain, return any `i` such that `A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]`.

<!--more-->

**Example 1:**

```
Input: [0,1,0]
Output: 1
```

**Example 2:**

```
Input: [0,2,1,0]
Output: 1
```

**Note:**

1. `3 <= A.length <= 10000`
2. `0 <= A[i] <= 10^6`
3. A is a mountain, as defined above.

#### 解题思路

&emsp;&emsp;直接遍历。。。

#### 解题代码

```c++
class Solution {
public:
    int peakIndexInMountainArray(vector<int>& A) {
        if (A.size() < 3) return 0;
        for (int i = 1; i < A.size(); ++i) {
            if (A[i] < A[i-1]) return i-1;
        }
        return 0;
    }
};
```

