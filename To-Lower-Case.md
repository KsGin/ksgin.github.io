---
title: To-Lower-Case
date: 2018-07-13 18:53:34
tags: LeetCode
---

#### 题目地址

[LeetCode#709 To Lower Case](https://leetcode.com/problems/to-lower-case/description/)

#### 题目描述

&emsp;&emsp;Implement function ToLowerCase() that has a string parameter str, and returns the same string in lowercase.

<!--more-->

#### 解题思路

&emsp;&emsp;实现 ToLowerCase() 方法。

#### 解题代码

```c++
class Solution {
public:
    string toLowerCase(string str) {
        for (auto &c : str) {
            if (c >= 'A' && c <= 'Z') {
                c += 32;
            }
        }
        return str;
    }
};
```

