---
title: Is-Subsequence
date: 2017-12-18 10:25:12
tags: LeetCode
---

[LeetCode#392 Is Subsequence](https://leetcode.com/problems/is-subsequence/description/)

&emsp;&emsp;Given a string **s** and a string **t**, check if **s** is subsequence of **t**.

You may assume that there is only lower case English letters in both **s** and **t**. **t** is potentially a very long (length ~= 500,000) string, and **s**is a short string (<=100).

A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (ie, `"ace"` is a subsequence of `"abcde"` while `"aec"` is not).

**Example 1:**
**s** = `"abc"`, **t** = `"ahbgdc"`

Return `true`.

**Example 2:**
**s** = `"axc"`, **t** = `"ahbgdc"`

Return `false`.

**Follow up:**
If there are lots of incoming S, say S1, S2, ... , Sk where k >= 1B, and you want to check one by one to see if T has its subsequence. In this scenario, how would you change your code?

<!--more-->

#### 解题思路

&emsp;&emsp;这道题是一道`Medium`，但是却非常的简单，定义指向S和T的指针，当他们所对应的字符相等时，两个指针都自增1，当他们不相等的时候T的指针自增。最后判断S的指针是否走到了最后就可以了。

#### 解题代码【.CPP】

```c++
//runtime:72ms
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int idxs = 0 , idxt = 0;
        while (idxs < s.size() && idxt < t.size()){
            if(s[idxs] == t[idxt]){
                ++idxs;
                ++idxt;
            } else {
                ++idxt;
            }
        }
        return idxs == s.size();
    }
};
```



