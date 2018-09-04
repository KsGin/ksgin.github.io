---
title: Subsets
date: 2018-01-26 19:31:59
tags: LeetCode
---

#### 题目地址

[LeetCode#78 Subsets](https://leetcode.com/problems/subsets/description/)

#### 题目描述

Given a set of **distinct** integers, *nums*, return all possible subsets (the power set).

**Note:** The solution set must not contain duplicate subsets.

<!--more-->

For example,
If **nums** = `[1,2,3]`, a solution is:

```
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

#### 解题思路

&emsp;&emsp;一道Medium题，但是题目很简单，就是输出给定集合的子集，解法也很简单，就是遍历填值。即我们主循环里遍历给定集合，子循环遍历结果集（二维集合），将其中的一维集合提取出来给他加上当前遍历的给定集合的值，然后再添加到结果集末尾。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    vector<vector<int>> subsets(vector<int> &nums) {
        vector<vector<int>> ret(0);
        ret.push_back(vector<int>());
        for (int i = 0; i < nums.size(); i++) {
            auto se = ret.size();
            for (auto j = 0; j < se; ++j) {
                auto tmp = ret[j];
                tmp.push_back(nums[i]);
                ret.push_back(tmp);
            }
        }
        return ret;
    }
};
```

