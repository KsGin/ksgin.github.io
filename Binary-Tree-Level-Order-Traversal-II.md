---
title: Binary-Tree-Level-Order-Traversal-II
date: 2018-03-27 14:16:59
tags: LeetCode
---

#### 题目地址

[LeetCode#107 Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/description/)

&emsp;&emsp;简直了，今天一进 LeetCode 网站，就弹出提示让我迁移到 LeetCode 国区，没想到竟然有中文版了？？？（而且中文版名字贼丑，力扣）是不是这是最后一个在 LeetCode 外区做的题了？

#### 题目描述

&emsp;&emsp;Given a binary tree, return the *bottom-up level order* traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).

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

return its bottom-up level order traversal as:

```
[
  [15,7],
  [9,20],
  [3]
]
```

#### 解题思路

&emsp;&emsp;这道题就是换了个存储方法而已，和之前的那道 [Binary Tree Level Order Traversal](https://blog.ksgin.com/2018/03/24/binary-tree-level-order-traversal/#more)没啥区别。

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
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> res(0);
        if(!root) return res;
        res.insert(res.begin(),vector<int>(0));
        queue<TreeNode*> q;
        q.push(root);
        q.push(nullptr);
        while (q.size() > 1){
            auto p = q.front();
            q.pop();
            if (!p) {
                res.insert(res.begin(), vector<int>(0));
                q.push(nullptr);
            } else {
                if(p->left) q.push(p->left);
                if(p->right) q.push(p->right);
                res[0].push_back(p->val);
            }
        }
        return res;
    }
};
```

