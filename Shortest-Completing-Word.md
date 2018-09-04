---
title: Shortest-Completing-Word
date: 2018-02-16 19:32:57
tags: LeetCode
---

#### 题目地址

[LeetCode#748 Shortest Completing Word](https://leetcode.com/problems/shortest-completing-word/description/)

#### 题目描述

&emsp;&emsp;Find the minimum length word from a given dictionary `words`, which has all the letters from the string `licensePlate`. Such a word is said to *complete* the given string `licensePlate`

&emsp;&emsp;Here, for letters we ignore case. For example, `"P"` on the `licensePlate` still matches `"p"` on the word.

&emsp;&emsp;It is guaranteed an answer exists. If there are multiple answers, return the one that occurs first in the array.

&emsp;&emsp;The license plate might have the same letter occurring multiple times. For example, given a `licensePlate` of `"PP"`, the word `"pair"` does not complete the `licensePlate`, but the word `"supper"` does.

<!--more-->

**Example 1:**

```
Input: licensePlate = "1s3 PSt", words = ["step", "steps", "stripe", "stepple"]
Output: "steps"
Explanation: The smallest length word that contains the letters "S", "P", "S", and "T".
Note that the answer is not "step", because the letter "s" must occur in the word twice.
Also note that we ignored case for the purposes of comparing whether a letter exists in the word.
```

**Example 2:**

```
Input: licensePlate = "1s3 456", words = ["looks", "pest", "stew", "show"]
Output: "pest"
Explanation: There are 3 smallest length words that contains the letters "s".
We return the one that occurred first.
```

**Note:**

1. `licensePlate` will be a string with length in range `[1, 7]`.
2. `licensePlate` will contain digits, spaces, or letters (uppercase or lowercase).
3. `words` will have a length in the range `[10, 1000]`.
4. Every `words[i]` will consist of lowercase letters, and have length in range `[1, 15]`.

#### 解题思路

&emsp;&emsp;按照题意，在words里找出一个能匹配licensePlate且长度最短的单词（多个答案选第一个），匹配的过程用map即可实现，刚开始我想的是先对words进行排序，碰到匹配成功的就可以直接返回了，不过不知道为什么一直有问题（我明明没错啊！），就改成了用变量进行保存迭代实现。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    bool judge(string licensePlate, string word) {
        unordered_map<char, int> um;
        for (auto c : word) {
            c = tolower(c);
            if (c <= 'z' && c >= 'a') {
                if (um[c] > 0)
                    ++um[c];
                else um[c] = 1;
            }
        }
        for (auto c : licensePlate) {
            c = tolower(c);
            if (c <= 'z' && c >= 'a') {
                if (um[c] > 0)
                    --um[c];
                else return false;
            }
        }
        return true;
    }

    string shortestCompletingWord(string licensePlate, vector<string> words) {
        string ret = "";
        for (auto w : words) {
            if (judge(licensePlate, w) && (ret.empty() || w.size() < ret.size()))
                ret = w;
        }
        return ret;
    }
};
```

