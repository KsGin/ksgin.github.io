---
title: Binary-Tree-Level-Order-Traversal
date: 2018-03-24 00:09:03
tags: LeetCode
---

#### 题目地址

[LeetCode#102 Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)

#### 题目描述

&emsp;&emsp;Given a binary tree, return the *level order* traversal of its nodes' values. (ie, from left to right, level by level).

<!--more-->

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

return its level order traversal as:

```
[
  [3],
  [9,20],
  [15,7]
]
```

#### 解题思路

&emsp;&emsp;树的层次遍历，队列实现。

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
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res(0);
        if(!root) return res;
        res.push_back(vector<int>(0));
        queue<TreeNode*> q;
        q.push(root);
        q.push(nullptr);
        while (q.size() > 1){
            auto p = q.front();
            q.pop();
            if (!p) {
                res.push_back(vector<int>(0));
                q.push(nullptr);
            } else {
                if(p->left) q.push(p->left);
                if(p->right) q.push(p->right);
                res[res.size()-1].push_back(p->val);
            }
        }
        return res;
    }
};
```

