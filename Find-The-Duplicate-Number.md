---
title: Find-The-Duplicate-Number
date: 2018-01-01 18:29:16
tags: LeetCode
---

[LeetCode#287 Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/description/)

&emsp;&emsp;Given an array *nums* containing *n* + 1 integers where each integer is between 1 and *n* (inclusive), prove that at least one duplicate number must exist. Assume that there is only one duplicate number, find the duplicate one.

<!--more-->

**Note:**

1. You **must not** modify the array (assume the array is read only).
2. You must use only constant, *O*(1) extra space.
3. Your runtime complexity should be less than `O(n2)`.
4. There is only one duplicate number in the array, but it could be repeated more than once.

#### 解题思路

&emsp;&emsp;这道题是可以用二分法进行查找的，即找到Mid，然后如果大于mid的数字的个数大于小于mid的数字的个数，就说明重复的数字出现在大于mid的部分。反之出现在小于mid的部分。如果相等，则输出Mid。

&emsp;&emsp;但是这里有一个很精妙的解法。我也是看别人的，假设数组中没有重复，那我们可以做到这么一点，就是将数组的下标和1到n每一个数一对一的映射起来。比如数组是`213`,则映射关系为`0->2, 1->1, 2->3`。假设这个一对一映射关系是一个函数f(n)，其中n是下标，f(n)是映射到的数。如果我们从下标为0出发，根据这个函数计算出一个值，以这个值为新的下标，再用这个函数计算，以此类推，直到下标超界。实际上可以产生一个类似链表一样的序列。比如在这个例子中有两个下标的序列，`0->2->3`。

&emsp;&emsp;但如果有重复的话，这中间就会产生多对一的映射，比如数组`2131`,则映射关系为`0->2, {1，3}->1, 2->3`。这样，我们推演的序列就一定会有环路了，这里下标的序列是`0->2->3->1->1->1->1->...`，而环的起点就是重复的数。

&emsp;&emsp;所以该题实际上就是找环路起点的题，和[Linked List Cycle II](http://segmentfault.com/a/1190000003718848#articleHeader5)一样。我们先用快慢两个下标都从0开始，快下标每轮映射两次，慢下标每轮映射一次，直到两个下标再次相同。这时候保持慢下标位置不变，再用一个新的下标从0开始，这两个下标都继续每轮映射一次，当这两个下标相遇时，就是环的起点，也就是重复的数。对这个找环起点算法不懂的，请参考[Floyd's Algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Tortoise_and_hare)。

#### 解题代码【.CPP】

```c++
public class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0;
        int fast = 0;
        // 找到快慢指针相遇的地方
        do{
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while(slow != fast);
        int find = 0;
        // 用一个新指针从头开始，直到和慢指针相遇
        while(find != slow){
            slow = nums[slow];
            find = nums[find];
        }
        return find;
    }
}
```