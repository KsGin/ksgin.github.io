---
title: Self-Dividing-Numbers
date: 2018-01-11 11:21:44
tags: LeetCode
---

#### 题目地址

 [LeetCode#728 Self Dividing Numbers](https://leetcode.com/problems/self-dividing-numbers/description/)

#### 题目描述

&emsp;&emsp;A *self-dividing number* is a number that is divisible by every digit it contains.

&emsp;&emsp;For example, 128 is a self-dividing number because `128 % 1 == 0`, `128 % 2 == 0`, and `128 % 8 == 0`.

&emsp;&emsp;Also, a self-dividing number is not allowed to contain the digit zero.

&emsp;&emsp;Given a lower and upper number bound, output a list of every possible self dividing number, including the bounds if possible.

<!--more-->

**Example 1:**

```
Input: 
left = 1, right = 22
Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 12, 15, 22]
```

**Note:**

&emsp;&emsp;The boundaries of each input argument are `1 <= left <= right <= 10000`.

#### 解题思路

&emsp;&emsp;Easy题，唯一需要考虑的就是如何判断一个数字是不是*self-dividing number*，题目上说的很清楚，能够整除他每一位上的数字的数字是*self-dividing number*，那就直接提出来计算就可以了，看代码吧。

#### 解题代码【.CPP】

```c++
class Solution {
    bool isSelfDividingNumber(int number){
        bool res = true;
        int tol = number;
        while (tol > 0 && res){
            int tmp = tol % 10;
            res = tmp && (number % tmp == 0);
            tol = tol / 10;
        }
        return res;
    }

public:
    vector<int> selfDividingNumbers(int left, int right) {
        vector<int> res;
        for (int i = left; i <= right; ++i) {
            if(isSelfDividingNumber(i)) res.push_back(i);
        }
        return res;
    }
};
```

