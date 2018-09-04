---
title: Longest-Word-in-Dictionary-through-Deleting
date: 2018-01-07 16:02:23
tags: LeetCode
---

[LeetCode#524 Longest Word in Dictionary through Deleting](https://leetcode.com/problems/longest-word-in-dictionary-through-deleting/description/)

&emsp;&emsp;Given a string and a string dictionary, find the longest string in the dictionary that can be formed by deleting some characters of the given string. If there are more than one possible results, return the longest word with the smallest lexicographical order. If there is no possible result, return the empty string.

<!--more-->

**Example 1:**

```
Input:
s = "abpcplea", d = ["ale","apple","monkey","plea"]

Output: 
"apple"

```

**Example 2:**

```
Input:
s = "abpcplea", d = ["a","b","c"]

Output: 
"a"

```

**Note:**

1. All the strings in the input will only contain lower-case letters.
2. The size of the dictionary won't exceed 1,000.
3. The length of all the strings in the input won't exceed 1,000.

#### 解题思路

&emsp;&emsp;先将字典按照字符串长度（如果长度一样就按照字母大小排序）排序，然后挨个遍历。直接看代码吧

#### 解题代码【.CPP】

```c++
class Solution {
public:
    string findLongestWord(string s, vector<string> &d) {

        sort(d.begin(), d.end(),
             [](string a, string b) {
                 if (a.size() > b.size()) return true;
                 if (a.size() == b.size()) return a < b;
                 return false;
             });

        for (auto str : d) {
            int idx = 0;
            for (auto ch : s){
                if(ch == str[idx]) idx++;
                if(idx == str.size()) return str;
            }
        }
        return "";
    }
};
```