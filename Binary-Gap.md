---
title: Binary-Gap
date: 2018-07-19 19:50:22
tags: LeetCode
---

#### 题目地址

[LeetCode#868 Binary Gap](https://leetcode.com/problems/binary-gap/description/)

#### 题目描述

&emsp;&emsp;Given a positive integer `N`, find and return the longest distance between two consecutive 1's in the binary representation of `N`.

&emsp;&emsp;If there aren't two consecutive 1's, return 0.

<!--more-->


**Example 1:**

```
Input: 22
Output: 2
Explanation: 
22 in binary is 0b10110.
In the binary representation of 22, there are three ones, and two consecutive pairs of 1's.
The first consecutive pair of 1's have distance 2.
The second consecutive pair of 1's have distance 1.
The answer is the largest of these two distances, which is 2.
```

**Example 2:**

```
Input: 5
Output: 2
Explanation: 
5 in binary is 0b101.
```

**Example 3:**

```
Input: 6
Output: 1
Explanation: 
6 in binary is 0b110.
```

**Example 4:**

```
Input: 8
Output: 0
Explanation: 
8 in binary is 0b1000.
There aren't any consecutive pairs of 1's in the binary representation of 8, so we return 0.
```

 **Note:**

- `1 <= N <= 10^9`

#### 解题思路

&emsp;&emsp;题目要求找出给定值的二进制串中距离最远的两个 `1` ，并返回其距离。可以化为二进制串然后遍历也可以直接像我这种使用移位并且计算。

#### 解题代码

```c++
class Solution {
public:
    int binaryGap(int N) {
        if (N == 0)
            return 0;
        int ret = -1;
        int cot = 0;
        while (N > 0) {
            if (N & 1) {
                ret = ret > cot ? ret : cot;
                cot = 0;
            }
            N = N >> 1;
            cot += ret >= 0;
        }
        return ret > 0 ? ret : 0;
    }
};
```

