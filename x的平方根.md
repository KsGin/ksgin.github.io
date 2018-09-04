---
title: x的平方根
date: 2018-04-02 13:22:28
tags: LeetCode
---

#### 题目地址

[LeetCode#69 x的平方根](https://leetcode-cn.com/problems/sqrtx/description/)

#### 题目描述

&emsp;&emsp;实现 `int sqrt(int x)` 函数。

&emsp;&emsp;计算并返回 *x* 的平方根。

**&emsp;&emsp;x** 保证是一个非负整数。

<!--more-->

**案例 1:**

```
输入: 4
输出: 2
```

**案例 2:**

```
输入: 8
输出: 2
说明: 8 的平方根是 2.82842..., 由于我们想返回一个整数，小数部分将被舍去。
```

#### 解题思路

&emsp;&emsp;求整数平方根并且是向下取整，那么可以判断当 `i^2 <= x && (i+1)^2 > x` 的时候返回 i 。

#### 解题代码

```c++
class Solution {
public:
    int mySqrt(int x) {
        long i = 0;
        for (i = 1; i <= x; ++i) {
            if(i * i > x && (i-1) * (i-1) <= x) {
                return i-1;
            }
        }
        return i-1;
    }
};
```

