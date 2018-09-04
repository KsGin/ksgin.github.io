---
title: Toeplitz-Matrix
date: 2018-02-15 18:57:17
tags: LeetCode
---

#### 题目地址

[LeetCode#766 Toeplitz Matrix](https://leetcode.com/problems/toeplitz-matrix/description/)

#### 题目描述

A matrix is *Toeplitz* if every diagonal from top-left to bottom-right has the same element.

Now given an `M x N` matrix, return `True` if and only if the matrix is *Toeplitz*.

<!--more-->

**Example 1:**

```
Input: matrix = [[1,2,3,4],[5,1,2,3],[9,5,1,2]]
Output: True
Explanation:
1234
5123
9512

In the above grid, the diagonals are "[9]", "[5, 5]", "[1, 1, 1]", "[2, 2, 2]", "[3, 3]", "[4]", and in each diagonal all elements are the same, so the answer is True.
```

**Example 2:**

```
Input: matrix = [[1,2],[2,2]]
Output: False
Explanation:
The diagonal "[1, 2]" has different elements.
```

**Note:**

1. `matrix` will be a 2D array of integers.
2. `matrix` will have a number of rows and columns in range `[1, 20]`.
3. `matrix[i][j]` will be integers in range `[0, 99]`.

#### 解题思路

&emsp;&emsp;题目要求判断一个N x M的矩阵是否满足在一个斜线（斜线是从左上到右下）上的数字都相同的条件，直接按照题目要求进行判断就是了。

#### 解题代码【.CPP】

```c++
class Solution {
    bool Judge(vector<vector<int>> &matrix, int i, int j) {
        for (int x = i, y = j; x < matrix.size() && y < matrix[i].size(); ++x, ++y) {
            if (matrix[x][y] != matrix[i][j]) return false;
        }
        return true;
    }

public:
    bool isToeplitzMatrix(vector<vector<int>> &matrix) {
        if (matrix.empty()) return true;
        bool ret = true;
        for (int i = 0; i < matrix.size(); ++i) {
            if (!Judge(matrix, i, 0)) return false;
        }
        for (int j = 1; j < matrix[0].size(); ++j) {
            if (!Judge(matrix, 0, j)) return false;
        }
        return ret;
    }
};
```

