---
layout: post 
serial: "From brute force to ultimate optimization - Recursion"
title: "Recursion, divide and conquer, DP"
categories: Algorithm
comments: true
---

> This example will use combination of recursion tree, DP and divide and conquer to solve problem. the example is burst balloons.


## [Burst Balloons](https://leetcode.com/problems/burst-balloons/){:target="_blank"} 
leetcode [Burst Balloons](https://leetcode.com/problems/different-ways-to-add-parentheses/){:target="_blank"} 

Given n balloons, indexed from 0 to n-1. Each balloon is painted with a number on it represented by array nums. You are asked to burst all the balloons. If the you burst balloon i you will get nums[left] * nums[i] * nums[right] coins. Here left and right are adjacent indices of i. After the burst, the left and right then becomes adjacent.

Find the maximum coins you can collect by bursting the balloons wisely.

Note: 
(1) You may imagine nums[-1] = nums[n] = 1. They are not real therefore you can not burst them.
(2) 0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

Example:

Given [3, 1, 5, 8]

Return 167

```
    nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
   coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167
```

### Video explanation
<iframe width="560" height="315" src="https://www.youtube.com/embed/PkqqhoO22sw" frameborder="0" allowfullscreen></iframe>

## DFS with memorizing and Divide and Conquer O(n^3)
Recursion method is all about how many option you have in each step, if you have N option, then divide it into N sub problem.
To avoid shift or remove elemnt from arrary, Instead of that element first, we will burst that element last.

```java
public int maxCoins(int[] nums) {
    if (nums == null || nums.length == 0)
        return 0;
    int n = nums.length;
    int num[] = new int[n+2];
    num[0] = 1;
    for (int i = 0; i < n; i++)
        num[i+1] = nums[i];
    num[n+1] = 1;

    return dfsCache(num, 1, n);
}

private int dfsCache(int[] num, int L, int R) {
    int n = R-L+1+2;
    int[][] cache = new int[n][n];
    
    return dfsCache(num, L, R, cache);
}
private int dfsCache(int[] num, int L, int R, int[][] cache) {
    if (cache[L][R] != 0)
        return cache[L][R];
    int coins = 0;
    for (int i = L; i <= R; i++) {
        int l = dfsCache(num, L, i-1, cache);
        int r = dfsCache(num, i+1, R, cache);
        int val = num[L-1]*num[i]*num[R+1] + l + r;
        coins = Math.max(coins, val);
    }
    cache[L][R] = coins;
    return coins;
}
```

## Bottom up DP O(n^3) time complexity
```java
private int DP(int[] num, int L, int R) {
    int n = R-L+1;
    int[][] dp = new int[n+2][n+2];

    for (int len = L; len <= R; len++) {
        for (int l = 1; l <= n-len+1; l++) {
            int r = l + len-1;
            for (int i = l; i <= r; i++) {
                int val = num[l-1]*num[i]*num[r+1];
                int sum = val + dp[l][i-1] + dp[i+1][r];
                dp[l][r] = Math.max(dp[l][r], sum);
            }
        }
    }
    return dp[L][R];
}
```
