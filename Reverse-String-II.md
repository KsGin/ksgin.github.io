---
title: Reverse-String-II
date: 2018-01-06 13:34:42
tags: LeetCode
---

[LeetCode#541 Reverse String II](https://leetcode.com/problems/reverse-string-ii/description/)

&emsp;&emsp;Given a string and an integer k, you need to reverse the first k characters for every 2k characters counting from the start of the string. If there are less than k characters left, reverse all of them. If there are less than 2k but greater than or equal to k characters, then reverse the first k characters and left the other as original.

<!--more-->

**Example:**

```
Input: s = "abcdefg", k = 2
Output: "bacdfeg"

```

Restrictions:

1. The string consists of lower English letters only.
2. Length of the given string and k will in the range [1, 10000]

#### 解题思路

&emsp;&emsp;题目是给出一个字符串，再给出一个值k，让我们将k个数字为一组进行翻转。但是是将0-k，2k-3k，…。就是说将字符串K个数字分为一组，第一组翻转，第二组保持，第三组翻转，第四组保持这样。用一个循环就可以解决了，很简单的题。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    string reverseStr(string s, int k) {
        int idx = 0 ;
        string::iterator it = s.begin();
        while (idx < s.size()){
            if(idx + k > s.size()){
                reverse(it , it + s.size() - idx);
            } else {
                reverse(it , it+k);
            }
            idx += 2*k;
            it += 2*k;
        }
        return s;
    }
};
```

