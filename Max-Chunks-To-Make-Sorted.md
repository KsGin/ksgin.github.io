---
title: Max-Chunks-To-Make-Sorted
date: 2018-01-23 21:31:12
tags: LeetCode
---

#### 题目地址

[LeetCode#769 Max Chunks To Make Sorted](https://leetcode.com/problems/max-chunks-to-make-sorted/description/)

#### 题目描述

Given an array `arr` that is a permutation of `[0, 1, ..., arr.length - 1]`, we split the array into some number of "chunks" (partitions), and individually sort each chunk.  After concatenating them, the result equals the sorted array.

What is the most number of chunks we could have made?

<!--more-->

**Example 1:**

```
Input: arr = [4,3,2,1,0]
Output: 1
Explanation:
Splitting into two or more chunks will not return the required result.
For example, splitting into [4, 3], [2, 1, 0] will result in [3, 4, 0, 1, 2], which isn't sorted.
```

**Example 2:**

```
Input: arr = [1,0,2,3,4]
Output: 4
Explanation:
We can split into two chunks, such as [1, 0], [2, 3, 4].
However, splitting into [1, 0], [2], [3], [4] is the highest number of chunks possible.
```

**Note:**

- `arr` will have length in range `[1, 10]`.
- `arr[i]` will be a permutation of `[0, 1, ..., arr.length - 1]`.

#### 解题思路

&emsp;&emsp;这道题题意是将一个无序数组排列分割最大次使得每一次分割的子数组排序后再拼接起来使整个数组有序。

&emsp;&emsp;索引遍历数组，以索引为分割点，索引左部（包括本身）的最大值如果小于右部的最小值，说明这两个不在一个分组内，给res加一。

#### 解题代码【.CPP】

```c++
class Solution {
    int maxV(vector<int> num , int idx){
        int rs = num[0];
        for (int i = 0; i <= idx; ++i) {
            rs = max(rs ,num[i]);
        }
        return rs;
    }
public:
    int maxChunksToSorted(vector<int>& arr) {
        int size =static_cast<int>(arr.size()) , res = 1;
        int rmin = arr[size-1];
        for (int i = size-2; i >= 0; --i) {
            int lmax = maxV(arr , i);
            if(lmax < rmin) res++;
            if(arr[i] < rmin) rmin = arr[i];
        }
        return res;
    }
};
```

