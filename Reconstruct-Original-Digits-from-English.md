---
title: Reconstruct-Original-Digits-from-English
date: 2017-12-19 08:49:14
tags: LeetCode
---

[LeetCode#423 Reconstruct Original Digits from English](https://leetcode.com/problems/reconstruct-original-digits-from-english/description/)

&emsp;&emsp;Given a **non-empty** string containing an out-of-order English representation of digits `0-9`, output the digits in ascending order.

**Note:**

1. Input contains only lowercase English letters.
2. Input is guaranteed to be valid and can be transformed to its original digits. That means invalid inputs such as "abc" or "zerone" are not permitted.
3. Input length is less than 50,000.

<!--more-->

**Example 1:**

```
Input: "owoztneoer"

Output: "012"

```

**Example 2:**

```
Input: "fviefuro"

Output: "45"
```

#### 解题思路

&emsp;&emsp;刚开始看到题的时候想到了建立一个英文数字数组`vector<string> numStrings {"zero" , "one" , ... , "nine"}`，然后将输入的字符串按照字符与字符个数存入表中，再挨个按数字英文的方式取出来。思路是没错，不过忽略了顺序的问题。比如按照从0-9的顺序排列的话`"threetwoseven"`就会过不去，因为刚开始是满足one的全部字符的，但是在取出one后剩下的字符已经没法满足任何一个数了。当然回溯可以解决这个问题，不过还有另一个更有效的方法，就是更改数字顺序。让唯一存在（他后边的数字的英文不包含这个数字的所有字符），比如zero肯定是唯一的，因为'z'只有他一个有。找到一个不会产生歧义的顺序后，就可以了。在这里我找到的顺序是`0 , 2 , 6 , 4 , 5 , 1 , 8 , 7 , 3 , 9`，即`"zero" , "two" , "six" , "four" , "five" , "one" , "eight" , "seven" , "three" , "nine"`。

#### 解题代码【.CPP】

```c++
class Solution {
    bool constructOriginalDigits(unordered_map<char , int> &table , string number){
        for (auto s : number){
            if(table[s] > 0) continue;
            else return false;
        }
        for (auto s : number) --table[s];
        return true;
    }

public:
    string originalDigits(string s) {
        vector<string> numberStrings{"zero" , "two" , "six" , "four" , "five" ,
                               "one" , "eight" , "seven" , "three" , "nine"};
        vector<int> numers{0 , 2 , 6 , 4 , 5 , 1 , 8 , 7 , 3 , 9};
        unordered_map<char , int> table;
        for (auto c : s){
            if(table[c] >= 1) ++table[c];
            else table[c] = 1;
        }

        vector<int> res(0);
        string ret = "";
        int idx = 0;
        while (idx <= 9){
            if(constructOriginalDigits(table,numberStrings[idx])) res.push_back(numers[idx]);
            else ++idx;
        }
        sort(res.begin() , res.end());
        for (auto c : res) ret.push_back(static_cast<char>(c + '0'));
        return ret;
    }
};
```

