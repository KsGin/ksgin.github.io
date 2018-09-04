---
title: Binary-Tree-Pruning
date: 2018-06-27 23:59:38
tags: LeetCode
---

#### 题目地址

[LeetCode#814 Binary Tree Pruning](https://leetcode.com/problems/binary-tree-pruning/description/)

#### 题目描述

&emsp;&emsp;We are given the head node `root` of a binary tree, where additionally every node's value is either a 0 or a 1.

&emsp;&emsp;Return the same tree where every subtree (of the given tree) not containing a 1 has been removed.

&emsp;&emsp;(Recall that the subtree of a node X is X, plus every node that is a descendant of X.)

<!--more-->

```
Example 1:
Input: [1,null,0,0,1]
Output: [1,null,0,null,1]
 
Explanation: 
Only the red nodes satisfy the property "every subtree not containing a 1".
The diagram on the right represents the answer.
```

```
Example 2:
Input: [1,0,1,0,0,0,1]
Output: [1,null,1,null,1]
```

```
Example 3:
Input: [1,1,0,1,1,0,1,0]
Output: [1,1,0,1,1,null,1]
```

**Note:**

- The binary tree will have at most `100 nodes`.
- The value of each node will only be `0` or `1`.

#### 解题思路

&emsp;&emsp;题目要求去除值都为 0 的子树，照做就可以了。

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
    int getSubTreeValueSum(TreeNode *root) {
        if (!root) return 0;
        else return root->val + getSubTreeValueSum(root->left) + getSubTreeValueSum(root->right);
    }

public:
    TreeNode *pruneTree(TreeNode *root) {
        if (!root) return root;

        if (!getSubTreeValueSum(root->left)) {
            root->left = nullptr;
        }
        if (!getSubTreeValueSum(root->right)) {
            root->right = nullptr;
        }

        pruneTree(root->left);
        pruneTree(root->right);
        
        return root;
    }
};
```

