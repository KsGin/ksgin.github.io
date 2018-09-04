---
title: Longest-Repeating-Character-Replacement
date: 2018-03-16 10:16:30
tags:
	- LeetCode
---

#### 题目地址

[LeetCode#424 Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/description/)

#### 题目描述

&emsp;&emsp;Given a string that consists of only uppercase English letters, you can replace any letter in the string with another letter at most *k* times. Find the length of a longest substring containing all repeating letters you can get after performing the above operations.

<!--more-->

**Note:**
Both the string's length and *k* will not exceed 104.

**Example 1:**

```
Input:
s = "ABAB", k = 2

Output:
4

Explanation:
Replace the two 'A's with two 'B's or vice versa.
```

**Example 2:**

```
Input:
s = "AABABBA", k = 1

Output:
4

Explanation:
Replace the one 'A' in the middle with 'B' and form "AABBBBA".
The substring "BBBB" has the longest repeating letters, which is 4.
```

#### 解题思路

&emsp;&emsp;本题可以采用滑动窗口法求最大值。我们维护这一个子串，并保存有当前子串的初始地址，Start 。

&emsp;&emsp;在遍历时，当遍历到一个新的字符，给当前字符的计数器+1，此时判断如果当前字符的个数满足大于当前子串长度减去 k 值（可替换的字符数量），则更新结果值。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int characterReplacement(string s, int k) {
        vector<int> counts(26 , 0);
        int ret = 0, maxCount = 0, start = 0;
        for (int i = 0; i < s.size(); ++i) {
            maxCount = max(maxCount , ++counts[s[i]-'A']);
            while(i-start-maxCount+1 > k){
                --counts[s[start]-'A'];
                ++start;
            }
            ret = max(ret , i-start+1);
        }
        return ret;
    }
};
```

