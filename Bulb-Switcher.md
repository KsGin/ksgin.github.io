---
title: Bulb-Switcher
date: 2018-02-07 20:26:53
tags: LeetCode
---

#### 题目地址

[LeetCode#319 Bulb Switcher](https://leetcode.com/problems/bulb-switcher/description/)

#### 题目描述

There are *n* bulbs that are initially off. You first turn on all the bulbs. Then, you turn off every second bulb. On the third round, you toggle every third bulb (turning on if it's off or turning off if it's on). For the *i*th round, you toggle every *i* bulb. For the *n*th round, you only toggle the last bulb. Find how many bulbs are on after *n* rounds.

<!--more-->

**Example:**

```
Given n = 3. 

At first, the three bulbs are [off, off, off].
After first round, the three bulbs are [on, on, on].
After second round, the three bulbs are [on, off, on].
After third round, the three bulbs are [on, off, off]. 

So you should return 1, because there is only one bulb is on.
```

#### 解题思路

&emsp;&emsp;题目意思是给定n个灯泡，初始处于关闭状态，然后从1到n遍历（假定为i），每隔 i个就改变灯泡的状态。即遍历1到n，然后对所有编号为i的倍数的灯泡进行状态改变，最后求出遍历完后处在亮（On）的灯泡有几个。

&emsp;&emsp;刚开始直接根据题意两层循环，结果超时，最后仔细考虑后发现当某个数的因子有奇数个时，它所代表的灯泡最终会处于亮的状态。而某个数的因子是奇数，则代表这个数是一个完全平方数，所以题目变成了求1-n中有多少个完全平方数。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int bulbSwitch(int n) {
        int res = 0;
        while (res * res <= n) ++ res;
        return res;
    }
};
```

