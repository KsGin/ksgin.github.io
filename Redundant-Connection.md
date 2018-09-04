---
title: Redundant-Connection
date: 2018-02-12 18:20:11
tags: LeetCode
---

#### 题目地址

[LeetCode#684 Redundant Connection](https://leetcode.com/problems/redundant-connection/description/)

#### 题目描述

In this problem, a tree is an **undirected** graph that is connected and has no cycles.

The given input is a graph that started as a tree with N nodes (with distinct values 1, 2, ..., N), with one additional edge added. The added edge has two different vertices chosen from 1 to N, and was not an edge that already existed.

<!--more-->

The resulting graph is given as a 2D-array of `edges`. Each element of `edges` is a pair `[u, v]` with `u < v`, that represents an **undirected** edge connecting nodes `u` and `v`.

Return an edge that can be removed so that the resulting graph is a tree of N nodes. If there are multiple answers, return the answer that occurs last in the given 2D-array. The answer edge `[u, v]` should be in the same format, with `u < v`.

**Example 1:**

```
Input: [[1,2], [1,3], [2,3]]
Output: [2,3]
Explanation: The given undirected graph will be like this:
  1
 / \
2 - 3
```

**Example 2:**

```
Input: [[1,2], [2,3], [3,4], [1,4], [1,5]]
Output: [1,4]
Explanation: The given undirected graph will be like this:
5 - 1 - 2
    |   |
    4 - 3
```

**Note:**

The size of the input 2D-array will be between 3 and 1000.

Every integer represented in the 2D-array will be between 1 and N, where N is the size of the input array.

**Update (2017-09-26):**
We have overhauled the problem description + test cases and specified clearly the graph is an **undirected** graph. For the **directed**graph follow up please see **Redundant Connection II**). We apologize for any inconvenience caused.

#### 解题思路

&emsp;&emsp;[并查集算法(Union-Find)](https://zh.wikipedia.org/wiki/%E5%B9%B6%E6%9F%A5%E9%9B%86)

&emsp;&emsp;这道题使用我上边给出的并查集算法可以很简单的解决，我们建立一个足以囊括题目所给连通图所有点的数组，并且将其所有值赋值为-1。在这里我们使用N[a]=b来表示a与b的直接连通关系，那么当N[a] = b && N[b] = c时a与c也有间接的连通关系。

&emsp;&emsp;而这道题要求我们求出给定无向图的冗余边（即去掉此边后仍保持全连通），那么我们遍历给定的边集合，并以遍历到的边的两个点为起点查找其最远的连通点，如果最终边上两个点得到的最远的连通点相同说明这个边上的两个点在同一个连通分量上，即当前边是冗余边，将其返回。

#### 解题代码【.CPP】

```c++
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        vector<int> root(2001, -1);
        for (auto edge : edges) {
            int x = find(root, edge[0]), y = find(root, edge[1]);
            if (x == y) return edge;
            root[x] = y;
        }
        return {};
    }
    int find(vector<int>& root, int i) {
        while (root[i] != -1) {
            i = root[i];
        }
        return i;
    }
```

