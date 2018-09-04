---
title: Jewels-and-Stones
date: 2018-02-14 21:38:34
tags: LeetCode
---

#### 题目地址

[LeetCode#771 Jewels and Stones](https://leetcode.com/problems/jewels-and-stones/description/)

#### 题目描述

You're given strings `J` representing the types of stones that are jewels, and `S` representing the stones you have.  Each character in `S`is a type of stone you have.  You want to know how many of the stones you have are also jewels.

The letters in `J` are guaranteed distinct, and all characters in `J` and `S` are letters. Letters are case sensitive, so `"a"` is considered a different type of stone from `"A"`.

<!--more-->

**Example 1:**

```
Input: J = "aA", S = "aAAbbbb"
Output: 3
```

**Example 2:**

```
Input: J = "z", S = "ZZ"
Output: 0
```

**Note:**

- `S` and `J` will consist of letters and have length at most 50.
- The characters in `J` are distinct.

#### 解题思路

&emsp;&emsp;题目的意思就是说求J中的字符出现在S中的次数之和，直接两层循环都可以。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int numJewelsInStones(string J, string S) {
        int ret = 0;
        for (auto j : J) {
            for (auto s : S) {
                ret += j == s;
            }
        }
        return ret;
    }
};
```

