---
title: Flipping-an-Image
date: 2018-06-26 11:14:09
tags: LeetCode
---

#### 题目地址

[LeetCode#832 Flipping an Image](https://leetcode.com/problems/flipping-an-image/description/)

#### 题目描述

&emsp;&emsp;Given a binary matrix `A`, we want to flip the image horizontally, then invert it, and return the resulting image.

&emsp;&emsp;To flip an image horizontally means that each row of the image is reversed.  For example, flipping `[1, 1, 0]` horizontally results in `[0, 1, 1]`.

&emsp;&emsp;To invert an image means that each `0` is replaced by `1`, and each `1` is replaced by `0`. For example, inverting `[0, 1, 1]` results in `[1, 0, 0]`.

<!--more-->

**Example 1:**

```
Input: [[1,1,0],[1,0,1],[0,0,0]]
Output: [[1,0,0],[0,1,0],[1,1,1]]
Explanation: First reverse each row: [[0,1,1],[1,0,1],[0,0,0]].
Then, invert the image: [[1,0,0],[0,1,0],[1,1,1]]
```

**Example 2:**

```
Input: [[1,1,0,0],[1,0,0,1],[0,1,1,1],[1,0,1,0]]
Output: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
Explanation: First reverse each row: [[0,0,1,1],[1,0,0,1],[1,1,1,0],[0,1,0,1]].
Then invert the image: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
```

**Notes:**

- `1 <= A.length = A[0].length <= 20`
- `0 <= A[i][j] <= 1`

#### 解题思路

&emsp;&emsp;按照题目要求先 reverse 每一行，再取反即可

#### 解题代码

```c++
class Solution {
public:
    vector<vector<int>> flipAndInvertImage(vector<vector<int>> &A) {
        if (A.empty()) return A;
        vector<vector<int>> ret(A.size(), vector<int>(A[0].size(), 0));
        for (int i = 0; i < A.size(); ++i) {
            for (int j = A[i].size() - 1; j >= 0; --j) {
                ret[i][A[i].size() - j - 1] = !A[i][j];
            }
        }
        return ret;
    }
};
```

