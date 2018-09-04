---
title: Custom-Sort-String
date: 2018-03-02 21:17:24
tags: LeetCode
---

#### 题目地址

[LeetCode#791 Custom Sort String](https://leetcode.com/problems/custom-sort-string/description/)

#### 题目描述

&emsp;&emsp;`S` and `T` are strings composed of lowercase letters. In `S`, no letter occurs more than once.

&emsp;&emsp;`S` was sorted in some custom order previously. We want to permute the characters of `T` so that they match the order that `S` was sorted. More specifically, if `x` occurs before `y` in `S`, then `x` should occur before `y` in the returned string.

&emsp;&emsp;Return any permutation of `T` (as a string) that satisfies this property.

<!--more-->

```
Example :
Input: 
S = "cba"
T = "abcd"
Output: "cbad"
Explanation: 
"a", "b", "c" appear in S, so the order of "a", "b", "c" should be "c", "b", and "a". 
Since "d" does not appear in S, it can be at any position in T. "dcba", "cdba", "cbda" are also valid outputs.
```

**Note:**

- `S` has length at most `26`, and no character is repeated in `S`.
- `T` has length at most `200`.
- `S` and `T` consist of lowercase letters only.

#### 解题思路

&emsp;&emsp;题目要求对给定串T进行排序，排序规则为凡是在S中出现过的字母都按照S的顺序排序，未出现的随便。由此我们可以先将S保存下来，然后遍历T，如果在S中出现过则放在该在的位置，如果没有则直接添加到后边。

#### 解题代码

```c++
class Solution {
public:
    string customSortString(string S, string T) {
        string ret = S;
        unordered_map<char , int> um;
        for (auto c : S) um[c]++;
        for (auto c : T){
            auto tmp = find(ret.begin(),ret.end(),c);
            if (tmp == end(ret)) ret.push_back(c);
            else {
                if(um[c] < 1)
                    ret.insert(tmp,c);
                else um[c]--;
            }
        }
        return ret;
    }
};
```

