---
title: Diameter-of-Binary-Tree
date: 2017-12-17 12:14:43
tags: LeetCode
---

[LeetCode#543 Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/description/)

&emsp;&emsp;Given a binary tree, you need to compute the length of the diameter of the tree. The diameter of a binary tree is the length of the **longest** path between any two nodes in a tree. This path may or may not pass through the root.

**Example:**
Given a binary tree 

```
          1
         / \
        2   3
       / \     
      4   5    

```

Return **3**, which is the length of the path [4,2,1,3] or [5,2,1,3].

**Note:** The length of path between two nodes is represented by the number of edges between them.

<!--more-->

#### 解题思路

&emsp;&emsp;这道题是求二叉树的直径，及最长两点间距离。其实就是求每个子树的左右深度相加再加一，找到其中的最大值就可以了，代码很简单。不过有两点需要注意

   	1. 不一定就是根节点的左右子树深度之和最高
  2. 使用一个递归函数进行计算的时候注意区分求深度和直径的关系，在我的代码中使用min保存直径，返回值是当前子树的深度。

解题代码【.CPP】

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

    int minHeight(TreeNode* root , int& min){
        if(!root) return 0;
        int left = minHeight(root->left , min);
        int right = minHeight(root->right , min);
        min = max(min , left+right+1);
        return max(left , right) + 1;
    }


public:
    int diameterOfBinaryTree(TreeNode* root) {
        if(!root) return 0;
        int res = 0;
        minHeight(root,res);
        return res - 1;
    }
};
```

