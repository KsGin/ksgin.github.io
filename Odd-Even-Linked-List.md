---
title: Odd-Even-Linked-List
date: 2017-12-26 11:54:25
tags: LeetCode
---

[LeetCode#328 Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/description/)

&emsp;&emsp;Given a singly linked list, group all odd nodes together followed by the even nodes. Please note here we are talking about the node number and not the value in the nodes.

You should try to do it in place. The program should run in O(1) space complexity and O(nodes) time complexity.

<!--more-->

**Example:**
Given `1->2->3->4->5->NULL`,
return `1->3->5->2->4->NULL`.

**Note:**
The relative order inside both the even and odd groups should remain as it was in the input. 
The first node is considered odd, the second node even and so on ...

#### 解题思路

&emsp;&emsp;这道题的意思是把所有奇数位的数字移到所有偶数位数字前边（是索引奇数位偶数位的不是奇数数字偶数数字）。我们使用两个指针，一个指向当前奇数位，一个指向偶数，然后遍历链表，并将偶数指针的下一个元素移到奇数指针的后边就可以了。

#### 解题思路【.CPP】

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

    void addNode(ListNode *a, ListNode *b) { //把b的后边加到a后边
        ListNode *tmp = b->next;
        b->next = tmp->next;
        tmp->next = a->next;
        a->next = tmp;
    }

public:
    ListNode *oddEvenList(ListNode *head) {
        if(!head || !head->next || !head->next->next)
            return head;
        ListNode *odd = head;
        ListNode *even = head->next;
        while (odd && even){
            if(even->next) addNode(odd , even);
            odd = odd->next;
            even = even->next;
        }
        return head;
    }
};
```

