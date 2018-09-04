---
title: Letter-Case-Permutation
date: 2018-02-19 20:25:27
tags: LeetCode
---

#### 题目地址

[LeetCode#784 Letter Case Permutation](https://leetcode.com/problems/letter-case-permutation/description/)

#### 题目描述

&emsp;&emsp;Given a string S, we can transform every letter individually to be lowercase or uppercase to create another string.  Return a list of all possible strings we could create.

<!--more-->

```
Examples:
Input: S = "a1b2"
Output: ["a1b2", "a1B2", "A1b2", "A1B2"]

Input: S = "3z4"
Output: ["3z4", "3Z4"]

Input: S = "12345"
Output: ["12345"]
```

**Note:**

- `S` will be a string with length at most `12`.
- `S` will consist only of letters or digits.

#### 解题思路

&emsp;&emsp;这是一道Easy题，题目要求对给定字符串中所有的字母进行大小写转换，并且将通过转换得到的所有字符串都保留下来。

&emsp;&emsp;这种类似于排列的题毫无疑问直接用递归解，由于给定字符串包括数字与字母，所以我们递归的时候判断下当前字符是不是字母，是字母的话需要递归字母转换前和转换后的（就是说需要调用两次递归函数，转换前调用一次，转换后调用一次），不是的话直接调用递归函数就可以了。

#### 解题代码【.CPP】

```c++
class Solution {
    void push(string s, int idx, set<string> &set) {
        if (idx >= s.size())return;
        push(s, idx + 1, set);
        if (isdigit(s[idx])) return;
        auto tmp = s;
        tmp[idx] += tmp[idx] >= 'a' && tmp[idx] <= 'z' ? -32 : 32;
        set.insert(tmp);
        push(tmp, idx + 1, set);
    }

public:
    vector<string> letterCasePermutation(string S) {
        set<string> tmp;
        tmp.insert(S);
        push(S, 0, tmp);
        vector<string> ret(0);
        for (auto t : tmp) ret.insert(ret.begin(),t);
        return ret;
    }
};
```

