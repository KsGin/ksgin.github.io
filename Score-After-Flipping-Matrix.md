---
title: Score-After-Flipping-Matrix
date: 2018-07-07 00:25:32
tags: LeetCode
---

#### 题目地址

[LeetCode#861 Score After Flipping Matrix](https://leetcode.com/problems/score-after-flipping-matrix/description/)

#### 题目描述

&emsp;&emsp;We have a two dimensional matrix `A` where each value is `0` or `1`.

&emsp;&emsp;A move consists of choosing any row or column, and toggling each value in that row or column: changing all `0`s to `1`s, and all `1`s to `0`s.

&emsp;&emsp;After making any number of moves, every row of this matrix is interpreted as a binary number, and the score of the matrix is the sum of these numbers.

&emsp;&emsp;Return the highest possible score.

<!--more--> 


**Example 1:**

```
Input: [[0,0,1,1],[1,0,1,0],[1,1,0,0]]
Output: 39
Explanation:
Toggled to [[1,1,1,1],[1,0,0,1],[1,1,1,1]].
0b1111 + 0b1001 + 0b1111 = 15 + 9 + 15 = 39
```

**Note:**

1. `1 <= A.length <= 20`
2. `1 <= A[0].length <= 20`
3. `A[i][j]` is `0` or `1`.

#### 解题思路

&emsp;&emsp;贪心算法，我们知道二进制串中越高位的占的比重越大，所以我们可以直接从高到底来判断是否进行 toggling  操作。

&emsp;&emsp;首先为行，行中每个数字的权重不一样，从高往低，所以在行中我们只需要对第一位进行判断（即数组第一个数字），当他为0的时候我们要将他变为1 就需要 toggling ，这样可以使这一行数字的二进制串最大。

&emsp;&emsp;列中每一列的权重一样，所以我们需要判断这一列中1的个数和0的个数，当0的个数大于1的个数时我们进行 toggling 操作。

#### 解题代码

```c++
class Solution {
public:
    int matrixScore(vector<vector<int>> &A) {
        if (A.empty()) return 0;
        for (int i = 0; i < A.size(); ++i) {
            int flag = 0;
            for (int j = 0; j < A[i].size(); ++j) {
                if (A[i][0] == 0 || flag) {
                    A[i][j] = A[i][j] == 1 ? 0 : 1;
                    flag = 1;
                }
            }
        }

        for (int i = 1; i < A[0].size(); ++i) {
            int sum = 0;
            for (int j = 0; j < A.size(); ++j) {
                sum += A[j][i];
            }
            if (sum * 2 < A.size()) {
                for (int j = 0; j < A.size(); ++j) {
                    A[j][i] = A[j][i] == 1 ? 0 : 1;
                }
            }
        }

        int ret = 0;
        for (int i = 0; i < A.size(); ++i) {
            for (int j = 0; j < A[i].size(); ++j) {
                ret += A[i][j] * pow(2, A[i].size() - 1 - j);
            }
        }
        return ret;
    }
};
```

