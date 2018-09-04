---
title: Evaluate-Division
date: 2018-03-21 11:04:26
tags: LeetCode
---

#### 题目地址

[LeetCode#399 Evaluate Division](https://leetcode.com/problems/evaluate-division/description/)

#### 题目描述

&emsp;&emsp;Equations are given in the format `A / B = k`, where `A` and `B` are variables represented as strings, and `k` is a real number (floating point number). Given some queries, return the answers. If the answer does not exist, return `-1.0`.

<!--more-->

**Example:**
Given `a / b = 2.0, b / c = 3.0.` 
queries are: `a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .` 
return `[6.0, 0.5, -1.0, 1.0, -1.0 ].`

The input is: `vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries`, where `equations.size() == values.size()`, and the values are positive. This represents the equations. Return `vector<double>`.

According to the example above:

```
equations = [ ["a", "b"], ["b", "c"] ],
values = [2.0, 3.0],
queries = [ ["a", "c"], ["b", "a"], ["a", "e"], ["a", "a"], ["x", "x"] ]. 
```

&emsp;&emsp;The input is always valid. You may assume that evaluating the queries will result in no division by zero and there is no contradiction.

#### 解题思路

&emsp;&emsp;原解来自[这里](https://leetcode.com/problems/evaluate-division/discuss/88310/C++-0ms-23-lines-DFS-solution-with-comments) ，DFS 解法。

#### 解题代码

```c++
class Solution {
private:
    unordered_map<string, vector<pair<string, double>>> children;                               // adjacency list

    pair<bool, double> search(string& a, string& b, unordered_set<string>& visited, double val) {
        if (visited.count(a) == 0) {
            visited.insert(a);                                                                  // mark a as visited
            for (auto p : children[a]) {
                double temp = val * p.second;                                                   // potential result
                if (p.first == b) { return make_pair(true, temp); }                             // found result

                auto result = search(p.first, b, visited, temp);
                if (result.first) { return result; }
            }
        }
        return make_pair(false, -1.0);
    }

public:
    vector<double> calcEquation(vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries) {
        vector<double> ans;

        for (int i = 0; i < equations.size(); i++) {
            children[equations[i].first].push_back(make_pair(equations[i].second, values[i]));      // build graph list a->b
            children[equations[i].second].push_back(make_pair(equations[i].first, 1.0 / values[i]));// build graph list b->a
        }
        for (auto p : queries) {
            unordered_set<string> visited;                                                          // to record visited nodes
            // p.first == p.second is special case
            ans.push_back(p.first == p.second && children.count(p.first) ? 1.0 : search(p.first, p.second, visited, 1.0).second);
        }
        return ans;
    }
};
```

