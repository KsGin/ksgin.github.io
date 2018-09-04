---
title: Kth-Smallest-Element-in-a-Sorted-Matrix
date: 2017-12-11 12:38:20
tags: LeetCode
---

&emsp;&emsp;[LeetCode#378 Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description/)

&emsp;&emsp;Given a *n* x *n* matrix where each of the rows and columns are sorted in ascending order, find the kth smallest element in the matrix.

&emsp;&emsp;Note that it is the kth smallest element in the sorted order, not the kth distinct element.

**Example:**

```
matrix = [
   [ 1,  5,  9],
   [10, 11, 13],
   [12, 13, 15]
],
k = 8,

return 13.

```

**Note: **
You may assume k is always valid, 1 ≤ k ≤ n2.

<!--more-->

#### 解题思路

&emsp;&emsp;这是一道Medium题，但是却简单的有点不像话。题目要求是求一个每一行有序矩阵的第Kth数据，及求它第N大的数据。本来可能是需要使用数组存储每一行的当前指针balabala一大堆，但是其实只需要将所有值放到一个一维数组里边再对一维数组排序就可以直接取值了。

#### 解题代码（.CPP）

```c++
class Solution {
public:
    int kthSmallest(std::vector<std::vector<int>>& matrix, int k) {
        std::vector<int> mat(0);
        for (auto vec : matrix){
            for (auto vrs : vec){
                mat.push_back(vrs);
            }
        }
        std::sort(mat.begin() , mat.end());
        return mat[k-1];
    }
};
```

