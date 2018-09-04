---
title: Rotated-Digits【好久没写代码了】
date: 2018-03-01 19:34:05
tags: LeetCode
---

#### 题目地址

[LeetCode#788 Rotated Digits](https://leetcode.com/problems/rotated-digits/description/)

#### 题目描述

&emsp;&emsp;X is a good number if after rotating each digit individually by 180 degrees, we get a valid number that is different from X. A number is valid if each digit remains a digit after rotation. 0, 1, and 8 rotate to themselves; 2 and 5 rotate to each other; 6 and 9 rotate to each other, and the rest of the numbers do not rotate to any other number.

<!--more-->

&emsp;&emsp;Now given a positive number `N`, how many numbers X from `1` to `N` are good?

```
Example:
Input: 10
Output: 4
Explanation: 
There are four good numbers in the range [1, 10] : 2, 5, 6, 9.
Note that 1 and 10 are not good numbers, since they remain unchanged after rotating.
```

**Note:**

- N  will be in range `[1, 10000]`.

#### 解题思路

&emsp;&emsp;这道题要求找出从1到N中满足数字能够旋转180°且会变成另外一个有意义数字的数字的个数。

&emsp;&emsp;1. 能够旋转180°

&emsp;&emsp;2. 旋转后与当前数字不相等

&emsp;&emsp;我们观察得知，要满足能够旋转180°，那么数字中每一位都满足为可以旋转为另一个数字，即1，2，5，6，8，9，0，要满足旋转后与当前数字不相等则需要满足至少有一位数字属于2，5，6，9中的一个。即满足存在某一位的数字属于2，5，6，9且不能出现3，4，7。

#### 解题代码【.CPP】

```c++
class Solution {
    bool judge(int i) {
        bool isHaving = false;
        bool isNotHaving = true;
        while (i > 0) {
            int j = i % 10;
            if(!isHaving && (j == 2 || j == 5 || j == 6 || j == 9))
                isHaving = true;
            if(isNotHaving && (j == 3 || j == 4 || j == 7))
                isNotHaving = false;

            i = i / 10;
        }
        return isHaving && isNotHaving;
    }

public:
    int rotatedDigits(int N) {
        int ret = 0;
        for (int i = 2; i <= N; ++i) {
            ret += judge(i);
        }
        return ret;
    }
};
```

