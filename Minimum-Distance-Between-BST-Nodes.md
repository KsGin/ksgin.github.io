---
title: Minimum-Distance-Between-BST-Nodes
date: 2018-03-07 16:27:41
tags: LeetCode
---

#### 题目地址

[LeetCode#783 Minimum Distance Between BST Nodes](https://leetcode.com/problems/minimum-distance-between-bst-nodes/description/)

#### 题目描述

&emsp;&emsp;Given a Binary Search Tree (BST) with the root node `root`, return the minimum difference between the values of any two different nodes in the tree.

<!--more-->

**Example :**

```
Input: root = [4,2,6,1,3,null,null]
Output: 1
Explanation:
Note that root is a TreeNode object, not an array.

The given tree [4,2,6,1,3,null,null] is represented by the following diagram:

          4
        /   \
      2      6
     / \    
    1   3  

while the minimum difference in this tree is 1, it occurs between node 1 and node 2, also between node 3 and node 2.
```

**Note:**

1. The size of the BST will be between 2 and `100`.
2. The BST is always valid, each node's value is an integer, and each node's value is different.

#### 解题思路

&emsp;&emsp;刚开始想复杂了。。

&emsp;&emsp;直接将树的所有节点值存入数组并排序，然后找到相差最小的数字返回其差值。

#### 解题代码【.CPP】

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
    void getValuse(TreeNode* root , vector<int>& values ){
        if(!root) return;
        values.push_back(root->val);
        getValuse(root->left,values);
        getValuse(root->right,values);
    }

public:
    int minDiffInBST(TreeNode* root) {
        vector<int> values(0);
        getValuse(root , values);
        sort(values.begin() , values.end());
        int ret = INT_MAX;
        for (int i = 1; i < values.size(); ++i) {
            ret = min(ret , values[i] - values[i-1]);
        }
        return ret;
    }
};
```

