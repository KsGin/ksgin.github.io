---
title: Transpose-Matrix
date: 2018-07-14 13:32:56
tags: LeetCode
---

#### 题目地址

[LeetCode#867 Transpose Matrix](https://leetcode.com/problems/transpose-matrix/description/)

#### 题目描述

&emsp;&emsp;Given a matrix `A`, return the transpose of `A`.

&emsp;&emsp;The transpose of a matrix is the matrix flipped over it's main diagonal, switching the row and column indices of the matrix. 

<!--more-->

**Example 1:**

```
Input: [[1,2,3],[4,5,6],[7,8,9]]
Output: [[1,4,7],[2,5,8],[3,6,9]]
```

**Example 2:**

```
Input: [[1,2,3],[4,5,6]]
Output: [[1,4],[2,5],[3,6]] 
```

**Note:**

1. `1 <= A.length <= 1000`
2. `1 <= A[0].length <= 1000`

#### 解题思路

&emsp;&emsp;题目要求实现矩阵的转置，不知道矩阵转置可以看 [这里](https://en.wikipedia.org/wiki/Transpose) 

#### 解题代码

```c++
class Solution {
public:
    vector<vector<int>> transpose(vector<vector<int>> &A) {
        if (A.empty()) return A;
        int row = A.size(), col = A[0].size();
        vector<vector<int>> ret(col, vector<int>(row, 0));
        for (int i = 0; i < A.size(); ++i) {
            for (int j = 0; j < A[i].size(); ++j) {
                ret[j][i] = A[i][j];
            }
        }
        return ret;
    }
};
```



