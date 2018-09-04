---
title: Shopping-Offers
date: 2018-01-09 14:35:29
tags: LeetCode
---

[LeetCode#638 Shopping Offers](https://leetcode.com/problems/shopping-offers/description/)

&emsp;&emsp;In LeetCode Store, there are some kinds of items to sell. Each item has a price.

&emsp;&emsp;However, there are some special offers, and a special offer consists of one or more different kinds of items with a sale price.

<!--more-->

&emsp;&emsp;You are given the each item's price, a set of special offers, and the number we need to buy for each item. The job is to output the lowest price you have to pay for **exactly** certain items as given, where you could make optimal use of the special offers.

&emsp;&emsp;Each special offer is represented in the form of an array, the last number represents the price you need to pay for this special offer, other numbers represents how many specific items you could get if you buy this offer.

&emsp;&emsp;You could use any of special offers as many times as you want.

**Example 1:**

```
Input: [2,5], [[3,0,5],[1,2,10]], [3,2]
Output: 14
Explanation: 
There are two kinds of items, A and B. Their prices are $2 and $5 respectively. 
In special offer 1, you can pay $5 for 3A and 0B
In special offer 2, you can pay $10 for 1A and 2B. 
You need to buy 3A and 2B, so you may pay $10 for 1A and 2B (special offer #2), and $4 for 2A.

```

**Example 2:**

```
Input: [2,3,4], [[1,1,0,4],[2,2,1,9]], [1,2,1]
Output: 11
Explanation: 
The price of A is $2, and $3 for B, $4 for C. 
You may pay $4 for 1A and 1B, and $9 for 2A ,2B and 1C. 
You need to buy 1A ,2B and 1C, so you may pay $4 for 1A and 1B (special offer #1), and $3 for 1B, $4 for 1C. 
You cannot add more items, though only $9 for 2A ,2B and 1C.

```

**Note:**

1. There are at most 6 kinds of items, 100 special offers.
2. For each item, you need to buy at most 6 of them.
3. You are **not** allowed to buy more items than you want, even if that would lower the overall price.

#### 解题思路

&emsp;&emsp;这道题的题目意思是，给了我们商品单价，和一些优惠券（优惠券的格式是前N个数字代表第N个商品有几个，最后一个数字是总价钱），就是说给了我们单价和打包价格，然后还给了我们需要买的每个商品的个数。让我们找到一个最优解，即买到需要的商品花费的最少钱数。

&emsp;&emsp;这里我们首先以不使用优惠券所花费的钱数作为初始值，然后对所有优惠券做一个递归，最后更新结果（res）的值就可以了。看代码吧

#### 解题代码【.CPP】

```c++
class Solution {
    void recvShopping(vector<int>& price, const vector<vector<int>> special , vector<int> needs , int cut , int& res){
        for (int i = 0; i < special.size(); ++i) {
            bool isValid = true;
            for (int j = 0; j < special[i].size() - 1; ++j) {
                if(isValid && special[i][j] > needs[j]) isValid = false;
            }
            if(isValid){
                vector<int> tmpNeeds = needs;
                int tmpCut = cut;
                for (int j = 0; j < special[i].size()-1; ++j) {
                    needs[j] -= special[i][j];
                }
                cut += special[i][special[i].size()-1];
                recvShopping(price, special , needs , cut , res);
                needs = tmpNeeds;
                cut = tmpCut;
            }
        }
        for (int i = 0; i < needs.size(); ++i) {
            cut += needs[i] * price[i];
        }
        res = min(res , cut);
    }


public:
    int shoppingOffers(vector<int>& price, vector<vector<int>>& special, vector<int>& needs) {
        int res = 0;
        for (int i = 0; i < price.size(); ++i) {
            res += price[i] * needs[i];
        }
        recvShopping(price , special , needs , 0 , res);
        return res;
    }
};
```

