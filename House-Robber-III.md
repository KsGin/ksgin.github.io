---
title: House-Robber-III
date: 2017-12-28 21:17:14
tags: LeetCode
---

[LeetCode# House Robber III](https://leetcode.com/problems/house-robber-iii/description/)

&emsp;&emsp;The thief has found himself a new place for his thievery again. There is only one entrance to this area, called the "root." Besides the root, each house has one and only one parent house. After a tour, the smart thief realized that "all houses in this place forms a binary tree". It will automatically contact the police if two directly-linked houses were broken into on the same night.

Determine the maximum amount of money the thief can rob tonight without alerting the police.

<!--more-->

**Example 1:**

```
     3
    / \
   2   3
    \   \ 
     3   1

```

Maximum amount of money the thief can rob = 3 + 3 + 1 = 7.

**Example 2:**

```
     3
    / \
   4   5
  / \   \ 
 1   3   1

```

Maximum amount of money the thief can rob = 4+5 = 9.

#### 解题思路

&emsp;&emsp;这个比较精妙的解法由网友[edyyy](http://home.cnblogs.com/u/1090659/)提供，这里的helper函数返回当前结点为根结点的最大rob的钱数，里面的两个参数l和r表示分别从左子结点和右子结点开始rob，分别能获得的最大钱数。在递归函数里面，如果当前结点不存在，直接返回0。否则我们对左右子结点分别调用递归函数，得到l和r。另外还得到四个变量，ll和lr表示左子结点的左右子结点的最大rob钱数，rl和rr表示右子结点的最大rob钱数。那么我们最后返回的值其实是两部分的值比较，其中一部分的值是当前的结点值加上ll, lr, rl, 和rr这四个值，这不难理解，因为抢了当前的房屋，那么左右两个子结点就不能再抢了，但是再下一层的四个子结点都是可以抢的；另一部分是不抢当前房屋，而是抢其左右两个子结点，即l+r的值，返回两个部分的值中的较大值即可，参见代码如下：

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
public:
    int rob(TreeNode* root) {
        int l = 0, r = 0;
        return helper(root, l, r);
    }
private:
    int helper(TreeNode* node, int& l, int& r) {
        if (!node) return 0;
        int ll = 0, lr = 0, rl = 0, rr = 0;
        l = helper(node->left, ll, lr);
        r = helper(node->right, rl, rr);
        return max(node->val + ll + lr + rl + rr, l + r);
    }
};
```

