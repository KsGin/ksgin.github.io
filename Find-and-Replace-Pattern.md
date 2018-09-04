---
title: Find-and-Replace-Pattern
date: 2018-09-04 16:00:52
tags: LeetCode
---

#### 题目地址

[LeetCode#890 Find and Replace Pattern](https://leetcode.com/problems/find-and-replace-pattern/description/)

#### 题目描述

&emsp;&emsp;You have a list of `words` and a `pattern`, and you want to know which words in `words` matches the pattern.

&emsp;&emsp;A word matches the pattern if there exists a permutation of letters `p` so that after replacing every letter `x` in the pattern with `p(x)`, we get the desired word.

&emsp;&emsp;(*Recall that a permutation of letters is a bijection from letters to letters: every letter maps to another letter, and no two letters map to the same letter.*)

&emsp;&emsp;Return a list of the words in `words` that match the given pattern. 

&emsp;&emsp;You may return the answer in any order.

<!--more-->

 **Example 1:**

```
Input: words = ["abc","deq","mee","aqq","dkd","ccc"], pattern = "abb"
Output: ["mee","aqq"]
Explanation: "mee" matches the pattern because there is a permutation {a -> m, b -> e, ...}. 
"ccc" does not match the pattern because {a -> c, b -> c, ...} is not a permutation,
since a and b map to the same letter.
```

 **Note:**

- `1 <= words.length <= 50`
- `1 <= pattern.length = words[i].length <= 20`

#### 解题思路

&emsp;&emsp;判断字符串是否满足 pattern 只需要判断每个字母相对于前边的字母是否相等的规律与 pattern 能否一致就可以了。

#### 解题代码

```c++
class Solution {
public:
    vector<string> findAndReplacePattern(vector<string> &words, string pattern) {
        vector<string> res(0);
        bool isPattern;
        for (auto &word : words) {
            isPattern = true;
            for (int i = 1; i < pattern.size(); ++i) {
                for (int j = 0; j < i; ++j) {
                    if ((word[i] == word[j]) ^ (pattern[i] == pattern[j])) {
                        isPattern = false;
                        break;
                    }
                }
                if (!isPattern) break;
            }
            if (isPattern) res.push_back(word);
        }
        return res;
    }
};
```

