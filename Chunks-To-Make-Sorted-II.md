---
layout: max
title: Chunks-To-Make-Sorted-II
date: 2018-03-26 10:39:56
tags: LeetCode
---

#### 题目地址

[LeetCode#768 Max Chunks To Make Sorted II](https://leetcode.com/problems/max-chunks-to-make-sorted-ii/description/)

#### 题目描述

&emsp;&emsp;*This question is the same as "Max Chunks to Make Sorted" except the integers of the given array are not necessarily distinct, the input array could be up to length 2000, and the elements could be up to 10\**8.*

&emsp;&emsp;Given an array `arr` of integers (**not necessarily distinct**), we split the array into some number of "chunks" (partitions), and individually sort each chunk.  After concatenating them, the result equals the sorted array.

<!--more-->

&emsp;&emsp;What is the most number of chunks we could have made?

**Example 1:**

```
Input: arr = [5,4,3,2,1]
Output: 1
Explanation:
Splitting into two or more chunks will not return the required result.
For example, splitting into [5, 4], [3, 2, 1] will result in [4, 5, 1, 2, 3], which isn't sorted.
```

**Example 2:**

```
Input: arr = [2,1,3,4,4]
Output: 4
Explanation:
We can split into two chunks, such as [2, 1], [3, 4, 4].
However, splitting into [2, 1], [3], [4], [4] is the highest number of chunks possible.
```

**Note:**

- `arr` will have length in range `[1, 2000]`.
- `arr[i]` will be an integer in range `[0, 10**8]`.

#### 解题思路

&emsp;&emsp;题目要求我们将给定数组分块，满足每一块内部排序后连接在一起总数组有序。也就是说满足第  N 个块的最小数据大于第 N-1 块的最大数据。

&emsp;&emsp;我们维护一个当前块的最大值索引位置，然后遍历数组，当遍历到的数字大于当前最大的时候，我们遍历它后边的所有数字，只有后边的所有数字都大于我们所保存的当前最大值的时候，才可以算作新的块。可以给 chunkCount 加一。

&emsp;&emsp;可以优化的部分是当在后边能找到比保存的当前最大值小的数字时，我们可以更新保存的当前最大值，同时更新遍历的索引 i 。

#### 解题代码

```c++
class Solution {
public:
    int maxChunksToSorted(vector<int>& arr) {
        if (arr.empty()) return 0;
        int chunkCount = 1;
        int chunkMaxIdx = 0;
        for (int i = 1; i < arr.size(); ++i) {
            if (arr[i] >= arr[chunkMaxIdx]){
                int tMax = chunkMaxIdx;
                int j = i;
                for (; j < arr.size(); ++j) {
                    if (arr[j] > arr[tMax]) tMax = j;
                    if (arr[j] < arr[chunkMaxIdx]){
                        chunkMaxIdx = tMax;
                        i = j;
                        break;
                    }
                }
                if( j == arr.size() ) {
                    chunkCount++;
                    chunkMaxIdx = i;
                }
            }
        }
        return chunkCount;
    }
};
```

