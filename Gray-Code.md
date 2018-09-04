---
title: Gray-Code
date: 2018-03-22 11:40:00
tags: LeetCode
---

#### 题目地址

[LeetCode#89 Gray Code](https://leetcode.com/problems/gray-code/description/)

#### 题目描述

&emsp;&emsp;The gray code is a binary numeral system where two successive values differ in only one bit.

&emsp;&emsp;Given a non-negative integer *n* representing the total number of bits in the code, print the sequence of gray code. A gray code sequence must begin with 0.

<!--more-->

&emsp;&emsp;For example, given *n* = 2, return `[0,1,3,2]`. Its gray code sequence is:

```
00 - 0
01 - 1
11 - 3
10 - 2
```

**Note:**
For a given *n*, a gray code sequence is not uniquely defined.

For example, `[0,2,3,1]` is also a valid gray code sequence according to the above definition.

For now, the judge is able to judge based on one instance of gray code sequence. Sorry about that.

#### 解题思路

&emsp;&emsp;思路比较简单，直接遍历就可以了，看代码。

#### 解题代码

```c++
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> ret(0);
        ret.push_back(0);
        if (n == 0) return ret;
        ret.push_back(1);
        for (int i = 1; i < n; ++i) {
            int size = ret.size();
            for (int j = size - 1; j >= 0; --j) {
                ret.push_back(static_cast<int &&>(ret[j] + pow(2 , i)));
            }
        }
        return ret;
    }
};
```

