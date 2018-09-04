---
title: Convert-Sorted-Array-to-Binary-Search-Tree
date: 2018-01-28 21:00:01
tags: LeetCode
---

#### 题目地址

[LeetCode#108 Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/description/)

#### 题目描述

&emsp;&emsp;Given an array where elements are sorted in ascending order, convert it to a height balanced BST.

<!--more-->

&emsp;&emsp;For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of *every* node never differ by more than 1.

**Example:**

```
Given the sorted array: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5
```

#### 解题思路

&emsp;&emsp;构造二叉树的题，我们用递归来实现（其实大部分关于二叉树的题都是无脑递归就可以了），给定数组的最中间值为根，左边子数组为左子树，右边子数组为右子树。

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
    void helper(vector<int> &nums, int l, int r, TreeNode *node) {
        if (l < r) {
            int mid = (l + r) / 2;
            node->val = nums[mid];
            if(l < mid){
                node->left = new TreeNode(0);
                helper(nums,l , mid , node->left);
            }
            if(mid+1 < r){
                node->right = new TreeNode(0);
                helper(nums,mid+1 , r , node->right);
            }
        } else {
            node = nullptr;
        }
    }

public:
    TreeNode *sortedArrayToBST(vector<int> &nums) {
        if(nums.empty()) return nullptr;
        TreeNode *root = new TreeNode(0);
        helper(nums , 0 , static_cast<int>(nums.size()), root);
        return root;
    }
};
```

