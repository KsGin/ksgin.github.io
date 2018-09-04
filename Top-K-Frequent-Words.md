---
title: Top-K-Frequent-Words
date: 2018-09-03 11:35:53
tags: LeetCode
---

#### 题目地址

[LeetCode#692 Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/description/)

#### 题目描述

&emsp;&emsp;Given a non-empty list of words, return the *k* most frequent elements.

&emsp;&emsp;Your answer should be sorted by frequency from highest to lowest. If two words have the same frequency, then the word with the lower alphabetical order comes first.

<!--more-->

**Example 1:**

```
Input: ["i", "love", "leetcode", "i", "love", "coding"], k = 2
Output: ["i", "love"]
Explanation: "i" and "love" are the two most frequent words.
    Note that "i" comes before "love" due to a lower alphabetical order.
```

**Example 2:**

```
Input: ["the", "day", "is", "sunny", "the", "the", "the", "sunny", "is", "is"], k = 4
Output: ["the", "is", "sunny", "day"]
Explanation: "the", "is", "sunny" and "day" are the four most frequent words,
    with the number of occurrence being 4, 3, 2 and 1 respectively.
```

**Note:**

1. You may assume *k* is always valid, 1 ≤ *k* ≤ number of unique elements.
2. Input words contain only lowercase letters.

**Follow up:**

1. Try to solve it in *O*(*n* log *k*) time and *O*(*n*) extra space.

#### 解题思路

&emsp;&emsp;经典题型，使用字典保存字符串与次数键值对，然后使用有序序列统计即可。

#### 解题代码

```c++
class Solution {
public:
    vector<string> topKFrequent(vector<string>& words, int k) {
       unordered_map<string, int> mp;
        for(auto word:words) mp[word]++;
        set<pair<int, string>> mySet;
        for(auto item:mp)
        {
            mySet.insert(make_pair(0-item.second, item.first)); //let the most frequent become the smallest
        }
        auto it=mySet.begin();
        vector<string> res;
        for(int i=0; i<k; i++)
        {
            res.push_back((*it).second);
            it++;
        }
        return res;
    }
};
```

