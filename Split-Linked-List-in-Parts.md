---
title: Split-Linked-List-in-Parts
date: 2018-03-06 14:05:25
tags: LeetCode
---

#### 题目地址

[LeetCode#725 Split Linked List in Parts](https://leetcode.com/problems/split-linked-list-in-parts/description/)

#### 题目描述

&emsp;&emsp;Given a (singly) linked list with head node `root`, write a function to split the linked list into `k` consecutive linked list "parts".

&emsp;&emsp;The length of each part should be as equal as possible: no two parts should have a size differing by more than 1. This may lead to some parts being null.

<!--more-->

&emsp;&emsp;The parts should be in order of occurrence in the input list, and parts occurring earlier should always have a size greater than or equal parts occurring later.

&emsp;&emsp;Return a List of ListNode's representing the linked list parts that are formed.

&emsp;&emsp;Examples 1->2->3->4, k = 5 // 5 equal parts [ [1], [2], [3], [4], null ]

**Example 1:**

```
Input: 
root = [1, 2, 3], k = 5
Output: [[1],[2],[3],[],[]]
Explanation:
The input and each element of the output are ListNodes, not arrays.
For example, the input root has root.val = 1, root.next.val = 2, \root.next.next.val = 3, and root.next.next.next = null.
The first element output[0] has output[0].val = 1, output[0].next = null.
The last element output[4] is null, but it's string representation as a ListNode is [].
```

**Example 2:**

```
Input: 
root = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], k = 3
Output: [[1, 2, 3, 4], [5, 6, 7], [8, 9, 10]]
Explanation:
The input has been split into consecutive parts with size difference at most 1, and earlier parts are a larger size than the later parts.
```

**Note:**

+ The length of `root` will be in the range `[0, 1000]`.
+ Each value of a node in the input will be an integer in the range `[0, 999]`.
+ `k` will be an integer in the range `[1, 50]`.

#### 解题思路

&emsp;&emsp;题目要求将链表分割 K 条，并满足当链表长度 Len % K == N 的时候，则前 N 条子链的长度为 Len / K + 1，其他的长度为 Len / K。当某一个子链长度为0时，数组里存 NULL 。

#### 解题代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    vector<ListNode*> splitListToParts(ListNode* root, int k) {
        int sLength = 0;
        ListNode* tmp = root;
        while (tmp) {
            sLength++;
            tmp = tmp->next;
        }
        int eLen = sLength / k;
        int eRem = sLength % k;
        tmp = root;
        vector<ListNode*> ret(0);
        for (int i = 0; i < k; ++i) {
            if(tmp){
                if(eLen > 0){
                    auto eRoot = tmp;
                    auto eTmp = eRoot;
                    tmp = tmp->next;
                    for (int j = 1; j < eLen; ++j) {
                        eTmp->next = tmp;
                        tmp = tmp->next;
                        eTmp = eTmp->next;
                    }
                    if(eRem > 0){
                        eTmp->next = tmp;
                        eTmp = eTmp->next;
                        tmp = tmp->next;
                        eRem--;
                    }
                    eTmp->next = nullptr;
                    ret.push_back(eRoot);
                } else {
                    if(eRem > 0){
                        auto eTmp = tmp;
                        tmp = tmp->next;
                        eTmp->next = nullptr;
                        eRem--;
                        ret.push_back(eTmp);
                    }
                }
            } else {
                ret.push_back(0);
            }
        }
        return ret;
    }
};
```

