---
title: All-Paths-From-Source-to-Target
date: 2018-06-30 11:57:37
tags: LeetCode
---

#### 题目地址

[LeetCode#797 All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/description/)

#### 题目描述

&emsp;&emsp;Given a directed, acyclic graph of `N` nodes.  Find all possible paths from node `0` to node `N-1`, and return them in any order.

&emsp;&emsp;The graph is given as follows:  the nodes are 0, 1, ..., graph.length - 1.  graph[i] is a list of all nodes j for which the edge (i, j) exists.

<!--more-->

```
Example:
Input: [[1,2], [3], [3], []] 
Output: [[0,1,3],[0,2,3]] 
Explanation: The graph looks like this:
0--->1
|    |
v    v
2--->3
There are two paths: 0 -> 1 -> 3 and 0 -> 2 -> 3.
```

**Note:**

- The number of nodes in the graph will be in the range `[2, 15]`.
- You can print different paths in any order, but you should keep the order of nodes inside one path.

#### 解题思路

&emsp;&emsp;唯一需要注意的是题目所给的值是当前节点所能到达的下一个节点位置，然后用 DFS 求解就是。

#### 解题代码

```c++
class Solution {
public:
    vector<vector<int>> allPathsSourceTarget(vector<vector<int>> &graph) {
        vector<vector<int>> ret;
        vector<int> tmp;
        helper(graph , ret , tmp , 0);
        return ret;
    }

    void helper(vector<vector<int>> &graph, vector<vector<int>> &ret, vector<int> tout, int start) {
        tout.push_back(start);
        if (tout.back() == graph.size() - 1) {
            ret.push_back(tout);
            return;
        }
        for (auto it : graph[start]) {
            helper(graph, ret, tout, it);
        }
    }
};
```

