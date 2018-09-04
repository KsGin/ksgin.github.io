---
title: Couples-Holding-Hands
date: 2018-03-05 22:48:53
tags: LeetCode
---

#### 题目地址

[LeetCode#765 Couples Holding Hands](https://leetcode.com/problems/couples-holding-hands/description/)

#### 题目描述

&emsp;&emsp;N couples sit in 2N seats arranged in a row and want to hold hands. We want to know the minimum number of swaps so that every couple is sitting side by side. A *swap* consists of choosing **any** two people, then they stand up and switch seats.

<!--more-->

&emsp;&emsp;The people and seats are represented by an integer from `0` to `2N-1`, the couples are numbered in order, the first couple being `(0, 1)`, the second couple being `(2, 3)`, and so on with the last couple being `(2N-2, 2N-1)`.

&emsp;&emsp;The couples' initial seating is given by `row[i]` being the value of the person who is initially sitting in the i-th seat.

**Example 1:**

```
Input: row = [0, 2, 1, 3]
Output: 1
Explanation: We only need to swap the second (row[1]) and third (row[2]) person.
```

**Example 2:**

```
Input: row = [3, 2, 0, 1]
Output: 0
Explanation: All couples are already seated side by side.
```

**Note:**

1. `len(row)` is even and in the range of `[4, 60]`.
2. `row` is guaranteed to be a permutation of `0...len(row)-1`.

#### 解题思路

&emsp;&emsp;直接遍历，当碰到不是情侣的时候（即不满足 x , x+1 且 x % 2 == 0），就在数组里查找的满足的值直接与当前进行替换。即解决一对是一对的政策。（话说我一个单身狗为什么要考虑帮他们做这个）

#### 解题代码

```c++
class Solution {
public:
    int minSwapsCouples(vector<int> &row) {
        int ret = 0;
        for (int i = 0; i < row.size() - 1; i += 2) {
            if ((row[i] % 2 == 0 && row[i] + 1 != row[i+1]) ||
                (row[i] % 2 == 1 && row[i] - 1 != row[i+1])) {
                for (int j = i + 2; j < row.size(); ++j) {
                    if ((row[i] % 2 == 0 && row[i] + 1 == row[j]) ||
                        (row[i] % 2 == 1 && row[i] - 1 == row[j])) {
                        swap(row[i + 1], row[j]);
                        ret++;
                        break;
                    }
                }
            }
        }
        return ret;
    }
};
```

