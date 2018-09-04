---
title: Kth-Smallest-Element-in-a-BST
date: 2018-03-08 07:34:07
tags: LeetCode
---

#### 题目地址

[LeetCode#230 Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/description/)

#### 题目描述

&emsp;&emsp;Given a binary search tree, write a function `kthSmallest` to find the **k**th smallest element in it.

<!--more-->

**Note: **
&emsp;&emsp;You may assume k is always valid, 1 ≤ k ≤ BST's total elements.

**Follow up:**
&emsp;&emsp;What if the BST is modified (insert/delete operations) often and you need to find the kth smallest frequently? How would you optimize the kthSmallest routine?

#### 解题思路

&emsp;&emsp;题目要求找到树种第K个最小的值，可以将树的节点值保存到容器数组中，然后排序就可以了。emm

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
    void getValue(TreeNode* root , vector<int>& values){
        if(!root) return;
        values.push_back(root->val);
        getValue(root->left , values);
        getValue(root->right , values);
    }

public:
    int kthSmallest(TreeNode* root, int k) {
        vector<int> valuse(0);
        getValue(root , valuse);
        sort(valuse.begin(),valuse.end());
        return valuse[k-1];
    }
};
```

