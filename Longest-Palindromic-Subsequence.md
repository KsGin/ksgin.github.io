---
title: Longest-Palindromic-Subsequence
date: 2018-03-13 09:29:58
tags: LeetCode
---

#### 题目地址

[LeetCode#516 Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/description/)

#### 题目描述

&emsp;&emsp;Given a string s, find the longest palindromic subsequence's length in s. You may assume that the maximum length of s is 1000.

<!--more-->

**Example 1:**
Input: 

```
"bbbab"
```

Output: 

```
4
```

One possible longest palindromic subsequence is "bbbb".

**Example 2:**
Input:

```
"cbbd"
```

Output:

```
2
```

One possible longest palindromic subsequence is "bb".

#### 解题思路

&emsp;&emsp;最长回文子序列，和最长回文字符串不同的地方在于这个不要求连续，做法和最长回文字符串也不差多少。我们使用 DP ，递推公式如下：

+ s[i] == s[j] : dp[i]\[j] = dp[i+1]\[j-1] + 2
+ s[i] != s[j] : dp[i]\[j] = max(dp[i+1]\[j] , dp[i]\[j-1])

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        vector<vector<int>> dp(s.size() , vector<int>(s.size() , 0));
        for(int i = s.size() - 1 ; i >= 0 ; --i){
            dp[i][i] = 1;
            for (int j = i+1; j < s.size(); ++j) {
                if (s[i] == s[j]) dp[i][j] = dp[i+1][j-1] + 2;
                else dp[i][j] = max(dp[i+1][j] , dp[i][j-1]);
            }
        }
        return dp[0][s.size()-1];
    }
};
```

