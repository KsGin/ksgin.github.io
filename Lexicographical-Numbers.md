---
title: Lexicographical-Numbers
date: 2018-03-25 19:56:56
tags: LeetCode
---

#### 题目地址

[LeetCode#386 Lexicographical Numbers](https://leetcode.com/problems/lexicographical-numbers/description/)

#### 题目描述

&emsp;&emsp;Given an integer *n*, return 1 - *n* in lexicographical order.

&emsp;&emsp;For example, given 13, return: [1,10,11,12,13,2,3,4,5,6,7,8,9].

&emsp;&emsp;Please optimize your algorithm to use less time and space. The input size may be as large as 5,000,000.

<!--more-->

#### 解题思路

&emsp;&emsp;要求按照字典顺序对从 0-n 的数字进行排序，我试了一下先生成数字然后用 `sort()` 但是直接 TEL 了，所以只能在生成数字的时候就有序。那么就直接递归进行生成数字。

#### 解题代码

```c++
class Solution {
    void addNumber(int num, int target, vector<int> &res) {
        if (num > target) return;
        res.push_back(num);
        addNumber(num * 10, target, res);
        if (num % 10 == 9) return;
        addNumber(num + 1, target, res);
    }

public:
    vector<int> lexicalOrder(int n) {
        vector<int> res(0);
        addNumber(1 , n , res);
        return res;
    }
};
```

