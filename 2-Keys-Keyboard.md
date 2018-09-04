---
title: 2-Keys-Keyboard
date: 2017-12-16 18:50:36
tags: LeetCode
---

[LeetCode#650 2 Keys Keyboard](https://leetcode.com/problems/2-keys-keyboard/description/)

Initially on a notepad only one character 'A' is present. You can perform two operations on this notepad for each step:

1. `Copy All`: You can copy all the characters present on the notepad (partial copy is not allowed).
2. `Paste`: You can paste the characters which are copied **last time**.

<!--more-->

Given a number `n`. You have to get **exactly** `n` 'A' on the notepad by performing the minimum number of steps permitted. Output the minimum number of steps to get `n` 'A'.

**Example 1:**

```
Input: 3
Output: 3
Explanation:
Intitally, we have one character 'A'.
In step 1, we use Copy All operation.
In step 2, we use Paste operation to get 'AA'.
In step 3, we use Paste operation to get 'AAA'.

```

**Note:**

1. The `n` will be in the range [1, 1000].

#### 解题思路

&emsp;&emsp;这道题的目的是求在最少的操作次数下，可以从初始的一个'A'得到给定个数的'A'，操作有两种，一种是Copy All,另一种的Paste。首先分析下我们的操作，如果要求得到3个'A'，我们需要先Copy All一次，然后Paste两次。题目要求我们必须是Copy All而不能partial copy，也就是说我们**最后一次Copy后剪贴板里的A的个数必须可以被N除尽（N%t==0 //t为最后一次剪贴板里'A'的数量）**，之后我们需要连续Paste N/t 次就可以满足N个'A'的要求。这样我们将题目要求分成了两部分，第一个是求t是多少，第二个是求最少经过多少次操作可以到达t个'A'的要求。这很明显，第二个递归就可以了。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int minSteps(int n) {
        if (n == 1) return 0;
        int dn = n / 2;
        while (n % dn != 0) --dn;
        return n / dn + minSteps(dn);
    }
};
```



