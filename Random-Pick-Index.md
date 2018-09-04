---
title: Random-Pick-Index
date: 2017-12-29 22:24:43
tags: LeetCode
---

[LeetCode#398 Random Pick Index](https://leetcode.com/problems/random-pick-index/description/)

&emsp;&emsp;Given an array of integers with possible duplicates, randomly output the index of a given target number. You can assume that the given target number must exist in the array.

<!--more-->

**Note:**
&emsp;&emsp;The array size can be very large. Solution that uses too much extra space will not pass the judge.

**Example:**

```
int[] nums = new int[] {1,2,3,3,3};
Solution solution = new Solution(nums);

// pick(3) should return either index 2, 3, or 4 randomly. Each index should have equal probability of returning.
solution.pick(3);

// pick(1) should return 0. Since in the array only nums[0] is equal to 1.
solution.pick(1);
```

#### 解题思路

&emsp;&emsp;本来想在构造方法里建表的，但是note里说了

>Solution that uses too much extra space will not pass the judge.

&emsp;&emsp;只好每个都算一遍了，Solution里把传进来的数组存到类的类成员里，然后在pick方法里遍历数组，并把target出现的位置索引都记下来，随机输出一个就OK了

#### 解题代码【.CPP】

```c++
class Solution {
    vector<int> _nums;
public:
    Solution(vector<int> nums) {
        _nums = move(nums);
    }

    int pick(int target) {
        vector<int> idxs(0);
        for (int i = 0; i < _nums.size(); ++i) {
            if(target == _nums[i]) idxs.push_back(i);
        }
        return idxs[rand() % idxs.size()];
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(nums);
 * int param_1 = obj.pick(target);
 */
```

