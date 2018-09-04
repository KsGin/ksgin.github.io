---
title: Unique-Binary-Search-Trees
date: 2018-05-26 19:39:34
tags: LeetCode
---

#### 题目地址

[LeetCode#96 Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/description/)

#### 题目描述

&emsp;&emsp;Given *n*, how many structurally unique **BST's** (binary search trees) that store values 1 ... *n*?

<!--more-->

**Example:**

```
Input: 3
Output: 5
Explanation:
Given n = 3, there are a total of 5 unique BST's:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

#### 解题思路

&emsp;&emsp;[卡特兰数](https://zh.wikipedia.org/wiki/%E5%8D%A1%E5%A1%94%E5%85%B0%E6%95%B0) + DP 求解。

#### 解题代码

```c++
class Solution {
public:
    int numTrees(int n) {
        if (n <= 0)
            return 0;
        vector<int> res(n + 1, 0);
        res[0] = 1;
        res[1] = 1;
        for (int i = 2; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                res[i] += res[j] * res[i - j - 1];
            }
        }
        return res[n];
    }
};
```

