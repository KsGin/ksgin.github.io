---
title: Find-Smallest-Letter-Greater-Than-Target
date: 2018-01-12 20:07:13
tags: LeetCode
---

#### 题目地址

[LeetCode#744 Find Smallest Letter Greater Than Target](https://leetcode.com/problems/find-smallest-letter-greater-than-target/description/)

#### 题目描述

&emsp;&emsp;Given a list of sorted characters `letters` containing only lowercase letters, and given a target letter `target`, find the smallest element in the list that is larger than the given target.

&emsp;&emsp;Letters also wrap around. For example, if the target is `target = 'z'` and `letters = ['a', 'b']`, the answer is `'a'`.

<!--more-->

**Examples:**

```
Input:
letters = ["c", "f", "j"]
target = "a"
Output: "c"

Input:
letters = ["c", "f", "j"]
target = "c"
Output: "f"

Input:
letters = ["c", "f", "j"]
target = "d"
Output: "f"

Input:
letters = ["c", "f", "j"]
target = "g"
Output: "j"

Input:
letters = ["c", "f", "j"]
target = "j"
Output: "c"

Input:
letters = ["c", "f", "j"]
target = "k"
Output: "c"

```

**Note:**

1. `letters` has a length in range `[2, 10000]`.
2. `letters` consists of lowercase letters, and contains at least 2 unique letters.
3. `target` is a lowercase letter.

#### 解题思路

&emsp;&emsp;题目给的数组是已经排好序的，所以我们连排序都不用，直接遍历一遍，找到比target大的就可以赋值并且终止循环直接输出了，由于题目中说明了

> Letters also wrap around. For example, if the target is `target = 'z'` and `letters = ['a', 'b']`, the answer is `'a'`。

&emsp;&emsp;就是说如果没有比他大的，我们返回第一个值就可以了。所以我们将res的初始值就设置为数组的第一个值，这样遍历一遍后无论是否找到，res都是正确的值。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    char nextGreatestLetter(vector<char> &letters, char target) {
        char res = letters[0];
        for (auto r : letters) {
            if (r > target) {
                res = r;
                break;
            }
        }
        return res;
    }
};
```

