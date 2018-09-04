---
title: Number-of-Lines-To-Write-String
date: 2018-07-07 20:08:02
tags: LeetCode
---

#### 题目地址

[LeetCode#806 Number of Lines To Write String](https://leetcode.com/problems/number-of-lines-to-write-string/description/)

#### 题目描述

&emsp;&emsp;We are to write the letters of a given string `S`, from left to right into lines. Each line has maximum width 100 units, and if writing a letter would cause the width of the line to exceed 100 units, it is written on the next line. We are given an array `widths`, an array where widths[0] is the width of 'a', widths[1] is the width of 'b', ..., and widths[25] is the width of 'z'.

&emsp;&emsp;Now answer two questions: how many lines have at least one character from `S`, and what is the width used by the last such line? Return your answer as an integer list of length 2.

<!--more-->

```
Example :
Input: 
widths = [10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
S = "abcdefghijklmnopqrstuvwxyz"
Output: [3, 60]
Explanation: 
All letters have the same length of 10. To write all 26 letters,
we need two full lines and one line with 60 units.
```

```
Example :
Input: 
widths = [4,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10,10]
S = "bbbcccdddaaa"
Output: [2, 4]
Explanation: 
All letters except 'a' have the same length of 10, and 
"bbbcccdddaa" will cover 9 * 10 + 2 * 4 = 98 units.
For the last 'a', it is written on the second line because
there is only 2 units left in the first line.
So the answer is 2 lines, plus 4 units in the second line. 
```

**Note:**

- The length of `S` will be in the range [1, 1000].
- `S` will only contain lowercase letters.
- `widths` is an array of length `26`.
- `widths[i]` will be in the range of `[2, 10]`.

#### 解题思路

&emsp;&emsp;直接遍历给定字符串，按照题目要求就可以了。

#### 解题代码

```c++
class Solution {
public:
    vector<int> numberOfLines(vector<int> &widths, string S) {
        vector<int> ret(2, 0);
        if (S.empty()) return ret;
        for (auto &c : S) {
            int w = widths[c - 'a'];
            if (ret[1] + w > 100) {
                ++ret[0];
                ret[1] = w;
            } else {
                ret[1] += w;
            }
        }
        if (ret[1] != 0) ++ret[0];
        return ret;
    }
};
```

