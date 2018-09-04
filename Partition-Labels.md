---
title: Partition-Labels
date: 2018-01-17 20:46:27
tags: LeetCode
---

#### 题目地址

[LeetCode#763 Partition Labels](https://leetcode.com/problems/partition-labels/description/)

#### 题目描述

&emsp;&emsp;A string `S` of lowercase letters is given. We want to partition this string into as many parts as possible so that each letter appears in at most one part, and return a list of integers representing the size of these parts.

<!--more-->

**Example 1:**

```
Input: S = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation:
The partition is "ababcbaca", "defegde", "hijhklij".
This is a partition so that each letter appears in at most one part.
A partition like "ababcbacadefegde", "hijhklij" is incorrect, because it splits S into less parts.

```

**Note:**

1. `S` will have length in range `[1, 500]`.
2. `S` will consist of lowercase letters (`'a'` to `'z'`) only.

#### 解题思路

&emsp;&emsp;这道题是求分割这个字符串使得每一组的字符不出现在其他组中，且分割次数最多。

&emsp;&emsp;我在网上看到有的使用DP，但是有一个比较好的思路（最后发现这个思路有很多人用了，和我的只有小部分不同）。就是说我们记录下每个字符第一次出现的位置和最后一次出现的位置（比如a第一次出现在0，第二次出现在10，那么0和10肯定在一组里边），将其按照第一次出现的顺序进行排序。这样我们将第一个字母的首尾位置作为一组，如果其他字符第一次出现的位置在这个组里，我们更新这一组的尾（更新为较大的一个）。如果其他字符第一次出现的位置不在这个组（就是说大于这个组的尾位置），说明他是新的一组，记下其尾位置作为下一组的位置，看代码吧。

#### 解题代码【.CPP】

```c++
class Solution {
public:
    vector<int> partitionLabels(string S) {
        vector<pair<int,int>> cp(26 , pair<int,int>(S.size() + 1,S.size() + 1));
        for (int i = 0; i < S.size(); ++i) {
            cp[S[i]-'a'].second = i + 1;
        }
        for (int i = static_cast<int>(S.size() - 1); i >= 0; --i) {
            cp[S[i]-'a'].first = i + 1;
        }

        sort(cp.begin() , cp.end());

        vector<int> res(0);
        int start = 0;
        for (int i = 0; i < cp.size(); ++i) {
            if(cp[i].first == S.size() + 1 || cp[i].second == S.size() + 1){
                continue;
            }
            if(res.empty()) res.push_back(cp[i].second);
            if(cp[i].first < res[res.size()-1] + start){
                if(cp[i].second > res[res.size()-1] + start)
                    res[res.size()-1] = cp[i].second - start;
            }
            if(cp[i].first > res[res.size()-1] + start) {
                start += res[res.size()-1];
                res.push_back(cp[i].second - start);
            }
        }
        return res;
    }
};
```

