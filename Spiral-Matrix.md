---
title: Spiral-Matrix
date: 2018-04-03 15:53:27
tags: LeetCode
---

#### 题目地址

[LeetCode#54 Spiral Matrix](https://leetcode-cn.com/problems/spiral-matrix/description/)

#### 题目描述

&emsp;&emsp;给出一个 *m* x *n* 的矩阵（*m* 行, *n* 列），请按照顺时针螺旋顺序返回元素。

<!--more-->

&emsp;&emsp;例如，给出以下矩阵：

```
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
```

&emsp;&emsp;应该返回 `[1,2,3,6,9,8,7,4,5]`。

#### 解题思路

&emsp;&emsp;递归按照每一圈来取值。

#### 解题代码

```c++
class Solution {
    void spiral(const vector<vector<int>> &matrix, vector<int> &result, int idx) {
        int left = idx;
        int top = idx;
        int right = matrix[top].size() - 1 - idx;
        int bottom = matrix.size() - 1 - idx;

        if (right < left || bottom < top) return;

        if (right == left && bottom == top) {
            result.push_back(matrix[bottom][right]);
            return;
        }

        for (int i = left; i <= right; ++i) {    // A -> B
            result.push_back(matrix[top][i]);
        }
        for (int i = top + 1; i <= bottom; ++i) {    // B -> C
            result.push_back(matrix[i][right]);
        }
        for (int i = right - 1; i >= left; --i) {    // C -> D
            result.push_back(matrix[bottom][i]);
        }
        for (int i = bottom - 1; i > top; --i) {    // D -> A
            result.push_back(matrix[i][left]);
        }

        spiral(matrix, result, idx + 1);
    }

public:
    vector<int> spiralOrder(vector<vector<int>> &matrix) {
        vector<int> result(0);
        if (matrix.empty())
            return result;
        spiral(matrix, result, 0);
        return result;
    }
};
```

