---
title: Maximum-Product-of-Word-Lengths
date: 2017-12-13 10:52:43
tags: LeetCode
---

[LeetCode#318 Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/description/)

Given a string array `words`, find the maximum value of `length(word[i]) * length(word[j])` where the two words do not share common letters. You may assume that each word will contain only lower case letters. If no such two words exist, return 0.

<!--more-->

**Example 1:**

Given `["abcw", "baz", "foo", "bar", "xtfn", "abcdef"]`
Return `16`
The two words can be `"abcw", "xtfn"`.

**Example 2:**

Given `["a", "ab", "abc", "d", "cd", "bcd", "abcd"]`
Return `4`
The two words can be `"ab", "cd"`.

**Example 3:**

Given `["a", "aa", "aaa", "aaaa"]`
Return `0`
No such pair of words.

**Credits:**
Special thanks to [@dietpepsi](https://leetcode.com/discuss/user/dietpepsi) for adding this problem and creating all test cases.

#### 解题思路

&emsp;&emsp;嗯，又是一道骚操作题。我是卡在了最后大数据集的超时那里，苦思不得解，只能去参考网上大佬的做法，一看果然是骚操作。

&emsp;&emsp;首先分析下题，就是求数组中两个互不存在相同字母的字符串的长度之积，暴力的遍历判断其是否存在相同字符是根本不行的。还是来说说网上的骚操作吧。

&emsp;&emsp;由于题目上说明了只会出现小写的字母，也就是26个字母，而我们所用的int是32位的，所以可以用一个int来存储一个字符串的信息，就是用int的后26位来存储对应的字符是否出现，1为出现0为未出现。这样下来在判断是否有共同字符的时候只需要与一下就可以了。代码如下：

#### 解题代码【.CPP】

```c++
class Solution {
public:
    int maxProduct(vector<string>& words) {
        int res = 0;
        vector<int> mask(words.size(), 0);
        for (int i = 0; i < words.size(); ++i) {
            for (char c : words[i]) {
                mask[i] |= 1 << (c - 'a');
            }
            for (int j = 0; j < i; ++j) {
                if (!(mask[i] & mask[j])) {
                    res = max(res, int(words[i].size() * words[j].size()));
                }
            }
        }
        return res;
    }
};
```

