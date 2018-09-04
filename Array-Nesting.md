---
title: Array-Nesting
date: 2018-03-03 14:58:04
tags: LeetCode
---

#### 题目地址

[LeetCode#565 Array Nesting](https://leetcode.com/problems/array-nesting/description/)

#### 题目描述

&emsp;&emsp;A zero-indexed array A of length N contains all integers from 0 to N-1. Find and return the longest length of set S, where S[i] = {A[i], A[A[i]], A[A[A[i]]], ... } subjected to the rule below.

&emsp;&emsp;Suppose the first element in S starts with the selection of element A[i] of index = i, the next element in S should be A[A[i]], and then A[A[A[i]]]… By that analogy, we stop adding right before a duplicate element occurs in S.

<!--more-->

**Example 1:**

```
Input: A = [5,4,0,3,1,6,2]
Output: 6
Explanation: 
A[0] = 5, A[1] = 4, A[2] = 0, A[3] = 3, A[4] = 1, A[5] = 6, A[6] = 2.

One of the longest S[K]:
S[0] = {A[0], A[5], A[6], A[2]} = {5, 6, 2, 0}

```

**Note:**

1. N is an integer within the range [1, 20,000].
2. The elements of A are all distinct.
3. Each element of A is an integer within the range [0, N-1].

#### 解题思路

&emsp;&emsp;题目中定义集合S[i]= {A[i], A[A[i]], A[A[A[i]]], ... }，即以A[i]开头的满足A[i]，A[A[i]]...最终满足A[A[..A[i]]]=i的一个集合。求从S[0]到S[N-1]种集合大小最大的值。

&emsp;&emsp;刚开始我使用递归直接求从0到N-1每一个集合的大小进行比较，思路可行但是出现时间超限，于是定义了数组isVisted来判定当前值是否访问过，如果访问过说明从这里开始的集合已经被另一个集合包括了（即A[0]=1，那么S[1]一定是S[0]的子集）。

#### 解体代码【.CPP】

```c++
class Solution {
    int recv(int start , int current , vector<int>& nums , vector<bool>& isVisted){
        if(nums[current] == start) return 1;
        else {
            isVisted[current] = true;
            return 1 + recv(start , nums[current] , nums , isVisted);
        }
    }

public:
    int arrayNesting(vector<int>& nums) {
        int ret = 0;
        vector<bool> isVisted(nums.size() , false);
        for (int i = 0 ; i < nums.size() ; ++i){
            if(!isVisted[i])
                ret = max(ret , recv(nums[i] , nums[i] , nums , isVisted));
        }
        return ret;
    }
};
```

