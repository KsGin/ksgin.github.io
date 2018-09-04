---
title: Binary-Tree-Right-Side-View
date: 2018-03-23 10:07:00
tags: LeetCode
---

#### 题目地址

[LeetCode#199 Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/description/)

#### 题目描述

&emsp;&emsp;Given a binary tree, imagine yourself standing on the *right* side of it, return the values of the nodes you can see ordered from top to bottom.

<!--more-->

For example:
Given the following binary tree,

```
   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
```

You should return `[1, 3, 4]`.

#### 解题思路

&emsp;&emsp;这道题要求求出二叉树每一层最右边的值。那么我们只需要先序遍历，然后每一层依次更新就可以了。

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
    void recv(TreeNode *root, int level, vector<int> &res) {
        if (!root) return;
        if (level >= res.size()) res.push_back(root->val);
        else res[level] = root->val;
        recv(root->left, level + 1, res);
        recv(root->right, level + 1, res);
    }

public:
    vector<int> rightSideView(TreeNode *root) {
        vector<int> res(0);
        recv(root, 0, res);
        return res;
    }
};
```

