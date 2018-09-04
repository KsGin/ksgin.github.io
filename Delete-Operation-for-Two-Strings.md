---
title: Delete-Operation-for-Two-Strings
date: 2017-12-23 19:09:11
tags: LeetCode
---

[LeetCode#583 Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/description/)

&emsp;&emsp;Given two words *word1* and *word2*, find the minimum number of steps required to make *word1* and *word2* the same, where in each step you can delete one character in either string.

<!--more-->

**Example 1:**

```
Input: "sea", "eat"
Output: 2
Explanation: You need one step to make "sea" to "ea" and another step to make "eat" to "ea".

```

**Note:**

1. The length of given words won't exceed 500.
2. Characters in given words can only be lower-case letters.

#### 解题思路

&emsp;&emsp;要求他删除最少的字符使得这两个字符串相等。我们可以找到这个字符串最大的相同串。然后用两个字符串长度减去相同串长度*2。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n1 = word1.size(), n2 = word2.size();
        vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1, 0));
        for (int i = 1; i <= n1; ++i) {
            for (int j = 1; j <= n2; ++j) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return n1 + n2 - 2 * dp[n1][n2];
    }
};
```

