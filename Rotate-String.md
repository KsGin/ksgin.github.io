---
title: Rotate-String
date: 2018-04-10 12:53:32
tags: LeetCode
---

#### 题目地址

[LeetCode#796 Rotate String](https://leetcode.com/problems/rotate-string/description/)

#### 题目描述

&emsp;&emsp;We are given two strings, `A` and `B`.

&emsp;&emsp;A *shift on A* consists of taking string `A` and moving the leftmost character to the rightmost position. For example, if `A = 'abcde'`, then it will be `'bcdea'` after one shift on `A`. Return `True` if and only if `A` can become `B` after some number of shifts on `A`.

<!--more-->

```
Example 1:
Input: A = 'abcde', B = 'cdeab'
Output: true

Example 2:
Input: A = 'abcde', B = 'abced'
Output: false
```

**Note:**

- `A` and `B` will have length at most `100`.

#### 解题思路

&emsp;&emsp;题目要求我们判断字符串 B 是不是 字符串 A 的 Rotate String 。它的定义是将字符串左边的一部分移到右边组成新的字符串，例如：`'ABCDEF'` 我们将 `AB` 移到右边之后为 `CDEFAB` 。也就是说，字符串 B 如果是由字符串 A 的前半部（顺序不能变）移到后边而来，那么就满足要求。依此，我们可以想到，我们将两个 A 字符串连接在一起成为一个新的串 C ，即 `ABCDEFABCDEF` ，那么满足要求的串 B 自然属于这个 C 串的子串（因为 C 串中间部分既包含了 A 的后半部也包含了 A 的前半部）。

#### 解题代码

```c++
class Solution {
public:
    bool rotateString(string A, string B) {
        return (A.length() == B.length()) && ((A + A).find(B) != string::npos);
    }
};
```

