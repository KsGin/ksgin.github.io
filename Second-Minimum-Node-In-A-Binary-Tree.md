---
title: Second-Minimum-Node-In-A-Binary-Tree
date: 2018-05-21 23:56:03
tags: LeetCode
---

#### 题目地址

[LeetCode#671 Second Minimum Node In A Binary Tree](https://leetcode.com/problems/second-minimum-node-in-a-binary-tree/description/)

#### 题目描述

&emsp;&emsp;Given a non-empty special binary tree consisting of nodes with the non-negative value, where each node in this tree has exactly `two`or `zero` sub-node. If the node has two sub-nodes, then this node's value is the smaller value among its two sub-nodes.

<!--more-->

&emsp;&emsp;Given such a binary tree, you need to output the **second minimum** value in the set made of all the nodes' value in the whole tree.

&emsp;&emsp;If no such second minimum value exists, output -1 instead.

**Example 1:**

```
Input: 
    2
   / \
  2   5
     / \
    5   7

Output: 5
Explanation: The smallest value is 2, the second smallest value is 5.
```

**Example 2:**

```
Input: 
    2
   / \
  2   2

Output: -1
Explanation: The smallest value is 2, but there isn't any second smallest value.
```

#### 解题思路

&emsp;&emsp;题目要求得到二叉树中第二小的节点值 (Second Minimum Node) ，遍历就可以了。

#### 解题代码

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
    int fmin = INT_MAX;
    int smin = INT_MAX;
public:
    int findSecondMinimumValue(TreeNode* root) {
        if (!root) return smin;
        int val = root->val;
        if (val < fmin) {
            smin = fmin;
            fmin = val;
        } else if (val > fmin && val < smin){
            smin = val;
        }
        findSecondMinimumValue(root->left);
        findSecondMinimumValue(root->right);
        return smin == INT_MAX ? -1 : smin;
    }
};
```

