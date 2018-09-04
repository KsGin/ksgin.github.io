---
title: Max-Increase-to-Keep-City-Skyline
date: 2018-06-24 12:16:45
tags: LeetCode
---

#### 题目地址

[LeetCode#807 Max Increase to Keep City Skyline](https://leetcode.com/problems/max-increase-to-keep-city-skyline/description/)

#### 题目描述

&emsp;&emsp;In a 2 dimensional array `grid`, each value `grid[i][j]` represents the height of a building located there. We are allowed to increase the height of any number of buildings, by any amount (the amounts can be different for different buildings). Height 0 is considered to be a building as well. 

<!--more-->

&emsp;&emsp;At the end, the "skyline" when viewed from all four directions of the grid, i.e. top, bottom, left, and right, must be the same as the skyline of the original grid. A city's skyline is the outer contour of the rectangles formed by all the buildings when viewed from a distance. See the following example.

&emsp;&emsp;What is the maximum total sum that the height of the buildings can be increased?

```
Example:
Input: grid = [[3,0,8,4],[2,4,5,7],[9,2,6,3],[0,3,1,0]]
Output: 35
Explanation: 
The grid is:
[ [3, 0, 8, 4], 
  [2, 4, 5, 7],
  [9, 2, 6, 3],
  [0, 3, 1, 0] ]

The skyline viewed from top or bottom is: [9, 4, 8, 7]
The skyline viewed from left or right is: [8, 7, 9, 3]

The grid after increasing the height of buildings without affecting skylines is:

gridNew = [ [8, 4, 8, 7],
            [7, 4, 7, 7],
            [9, 4, 8, 7],
            [3, 3, 3, 3] ]
```

**Notes:**

- `1 < grid.length = grid[0].length <= 50`.
- All heights `grid[i][j]` are in the range `[0, 100]`.
- All buildings in `grid[i][j]` occupy the entire grid cell: that is, they are a `1 x 1 x grid[i][j]` rectangular prism.

#### 解题思路

&emsp;&emsp;直接先遍历找到每一行每一列的最大值，然后  `grid[i][j]`  所能增加的高度就是行最高和列最高中较小的一个减去 `grid[i][j]` 当前值，遍历求和即可。

#### 解题代码

```c++
class Solution {
public:
    int maxIncreaseKeepingSkyline(vector<vector<int>> &grid) {
        if (grid.empty())
            return 0;
        vector<int> rowMax(grid.size(), 0), colMax(grid[0].size(), 0);
        for (int i = 0; i < grid.size(); ++i) {
            for (int j = 0; j < grid[i].size(); ++j) {
                rowMax[i] = rowMax[i] > grid[i][j] ? rowMax[i] : grid[i][j];
                colMax[j] = colMax[j] > grid[i][j] ? colMax[j] : grid[i][j];
            }
        }

        int ret = 0;

        for (int i = 0; i < grid.size(); ++i) {
            for (int j = 0; j < grid[i].size(); ++j) {
                int minMax = rowMax[i] < colMax[j] ? rowMax[i] : colMax[j];
                int increase = minMax - grid[i][j];
                ret += increase > 0 ? increase : 0;
            }
        }

        return ret;
    }
};
```

