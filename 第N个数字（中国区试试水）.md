---
title: 第N个数字（中国区试试水）
date: 2018-03-27 22:28:52
tags: LeetCode
---

&emsp;&emsp;今天中午提交申请转移到中国区，今天下午就发邮件提示成功了，来试试水。不过感觉有点坑啊，后悔了。。。

#### 题目地址

[LeetCode#400 第N个数字](https://leetcode-cn.com/problems/nth-digit/description/)

#### 题目描述

&emsp;&emsp;在无限的整数序列 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...中找到第 *n* 个数字。

<!--more-->

**注意:**
*n* 是正数且在32为整形范围内 ( *n* < 231)。

**示例 1:**

```
输入:
3

输出:
3
```

**示例 2:**

```
输入:
11

输出:
0

说明:
第11个数字在序列 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ... 里是0，它是10的一部分。
```

#### 解题思路

&emsp;&emsp;这道题意思是把数字当字符串来处理，比如 12 相当于两个字符，最后查找第 N 个数字值。

&emsp;&emsp;我们可以很简单的想到，数字为 1 位的共有 9 个，数字为 2 位的有 90 个，3 位的有 900 个。所以可以先行判断数字落到的区间，最后查值就可以了。

#### 解题代码

```c++
class Solution {
public:
    int findNthDigit(int n) {
        long long len = 1, cnt = 9, start = 1;
        while (n > len * cnt) {
            n -= len * cnt;
            ++len;
            cnt *= 10;
            start *= 10;
        }
        start += (n - 1) / len;
        string t = to_string(start);
        return t[(n - 1) % len] - '0';
    }
};
```

