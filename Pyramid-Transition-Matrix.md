---
title: Pyramid-Transition-Matrix
date: 2018-02-04 12:05:31
tags: LeetCode
---

#### 题目地址

[LeetCode#756 Pyramid Transition Matrix](https://leetcode.com/problems/pyramid-transition-matrix/description/)

#### 题目描述

&emsp;&emsp;We are stacking blocks to form a pyramid. Each block has a color which is a one letter string, like `'Z'`.

<!--more-->

&emsp;&emsp;For every block of color `C` we place not in the bottom row, we are placing it on top of a left block of color `A` and right block of color `B`. We are allowed to place the block there only if `(A, B, C)` is an allowed triple.

&emsp;&emsp;We start with a bottom row of `bottom`, represented as a single string. We also start with a list of allowed triples `allowed`. Each allowed triple is represented as a string of length 3.

&emsp;&emsp;Return true if we can build the pyramid all the way to the top, otherwise false.

**Example 1:**

```
Input: bottom = "XYZ", allowed = ["XYD", "YZE", "DEA", "FFF"]
Output: true
Explanation:
We can stack the pyramid like this:
    A
   / \
  D   E
 / \ / \
X   Y   Z

This works because ('X', 'Y', 'D'), ('Y', 'Z', 'E'), and ('D', 'E', 'A') are allowed triples.
```

**Example 2:**

```
Input: bottom = "XXYX", allowed = ["XXX", "XXY", "XYX", "XYY", "YXZ"]
Output: false
Explanation:
We can't stack the pyramid to the top.
Note that there could be allowed triples (A, B, C) and (A, B, D) with C != D.
```

**Note:**

1. `bottom` will be a string with length in range `[2, 8]`.
2. `allowed` will have length in range `[0, 200]`.
3. Letters in all strings will be chosen from the set `{'A', 'B', 'C', 'D', 'E', 'F', 'G'}`.

#### 解题思路

&emsp;&emsp;DP

#### 解题代码【.CPP】

```c++
class Solution {
public:
    bool pyramidTransition(string bottom, vector<string>& allowed) {
        int n = static_cast<int>(bottom.size());
        unordered_map<string, vector<char>> mp;
        vector<vector<set<char>>> dp(n, vector<set<char>>(n));

        for(string temp: allowed){
            mp[temp.substr(0,2)].push_back(temp[2]);
        }

        for(int i = 0; i < n ; ++i) {
            dp[0][i].insert(bottom[i]);
        }

        for(int i = 1; i < n; ++i){
            for(int j = 0; j < n-i; ++j){
                for(char a: dp[i-1][j]){
                    for(char b: dp[i-1][j+1]){
                        string s = "";
                        s = s+a+b;
                        for(char c: mp[s]){
                            dp[i][j].insert(c);
                        }
                    }
                }
            }
        }

        return !dp[n-1][0].empty();
    }
};
```

