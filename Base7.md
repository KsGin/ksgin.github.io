---
title: Base7
date: 2017-12-30 10:54:04
tags: LeetCode
---

[LeetCode#504 Base7](https://leetcode.com/problems/base-7/description/)

&emsp;&emsp;Given an integer, return its base 7 string representation.

**Example 1:**

```c
Input: 100
Output: "202"
```

**Example 2:**

```c
Input: -7
Output: "-10"
```

**Note:** The input will be in range of [-1e7, 1e7].

<!--more-->

#### 解题思路

&emsp;&emsp;一道很简单的题，将十进制转换为7进制，只需要用输入数字不断除7，直到输入数字小于7为止，每次除的余数从低位到高位保存就是结果。直接看代码吧

#### 解题代码【.CPP】

```c++
class Solution {
public:
    string convertToBase7(int num) {
        bool isMinus = num < 0;
        num = abs(num);
        string ret = "";
        while (num >= 7) {
            ret = to_string(num % 7) + ret;
            num = num / 7;
        }
        ret = to_string(num) + ret;
        if (isMinus) ret = "-" + ret;
        return ret;
    }
};
```

