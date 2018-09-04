---
title: Flatten-Nested-List-Iterator
date: 2018-02-09 20:29:11
tags: LeetCode
---

#### 题目地址

[LeetCode#341 Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/description/)

#### 题目描述

Given a nested list of integers, implement an iterator to flatten it.

Each element is either an integer, or a list -- whose elements may also be integers or other lists.

<!--more-->

**Example 1:**
Given the list `[[1,1],2,[1,1]]`,

By calling *next* repeatedly until *hasNext* returns false, the order of elements returned by *next* should be: `[1,1,2,1,1]`.

**Example 2:**
Given the list `[1,[4,[6]]]`,

By calling *next* repeatedly until *hasNext* returns false, the order of elements returned by *next* should be: `[1,4,6]`.

#### 解题思路

&emsp;&emsp;这种题目根据题目要求做就行了。

#### 解题代码

```c++
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * class NestedInteger {
 *   public:
 *     // Return true if this NestedInteger holds a single integer, rather than a nested list.
 *     bool isInteger() const;
 *
 *     // Return the single integer that this NestedInteger holds, if it holds a single integer
 *     // The result is undefined if this NestedInteger holds a nested list
 *     int getInteger() const;
 *
 *     // Return the nested list that this NestedInteger holds, if it holds a nested list
 *     // The result is undefined if this NestedInteger holds a single integer
 *     const vector<NestedInteger> &getList() const;
 * };
 */
class NestedIterator {
    vector<int> list;
    int idx = 0;

    void _init_(const vector<NestedInteger> &nestedList){
        for (auto nest : nestedList) {
            if(nest.isInteger()) list.push_back(nest.getInteger());
            else _init_(nest.getList());
        }
    }

public:
    NestedIterator(vector<NestedInteger> &nestedList) {
        list = vector<int>(0);
        _init_(nestedList);
    }

    int next() {
        return list[idx++];
    }

    bool hasNext() {
        return idx < list.size();
    }
};

/**
 * Your NestedIterator object will be instantiated and called as such:
 * NestedIterator i(nestedList);
 * while (i.hasNext()) cout << i.next();
 */
```

