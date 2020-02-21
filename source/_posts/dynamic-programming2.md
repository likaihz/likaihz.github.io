---
title: 算法笔记：动态规划（2）
tags:
  - 算法
  - LeetCode
mathjax: true
date: 2020-02-21 20:26:36
---


## 问题类型

&#160; &#160; &#160; &#160;在[LeetCode](https://leetcode-cn.com/problemset/all/?topicSlugs=dynamic-programming)上目前共有200道左右的动态规划相关的题，根据现有题目可以总结出一些题型，熟悉题型以及相关的描述能够帮助我们更准确地判断动态规划使用的场景。

* 通向目标点的最小（最大）路径
* 不同方法数
* 区间合并
* 字符串上的动态规划
* 决策类

<!--more-->

## 通向目标点的最小（最大）路径

### 问题描述

&#160; &#160; &#160; &#160;这一类问题通常是给定一个目标点（目标状态），要求到达该目标点的代价最小（或最大）的路径的代价。一般来说都是求“代价”，而不是求“路径”。这种题的建模技巧在于，首先要明确“目标点”是什么，“代价”是什么，以及在通向该目标点的过程中经过的“中间状态”是哪些，dp[]数组就保存这些状态就可以。创建状态转移方程的时候通常是从前面的状态中选一个最大（最小）的再加上当前状态的权值。

### 例题

#### 最小路径和

> [(LeetCode.64)](https://leetcode-cn.com/problems/minimum-path-sum/)给定一个包含非负整数的 $m\times n$网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
> 说明：每次只能向下或者向右移动一步。
> 示例：
>>  输入:
>> [
>>   [1,3,1],
>>   [1,5,1],
>>   [4,2,1]
>> ]
>> 输出: 7
>> 解释: 因为路径 1→3→1→1→1 的总和最小。

&#160; &#160; &#160; &#160;此题的目标点很明确，就是grid[m-1][n-1]这个点，代价就是路径上经过的所有的点的权值，中间状态就是grid中的所有点，因为在走向目标点的过程中，所有其他的点都有可能经过，因此可以建立一个dp[m][n]的数组用以保存中间状态。

1. 确定状态：dp[i][j]表示的是到达$(i,j)$这个点的最小代价。
2. 边界值：本题的状态数组是二维的，边界值是两组，分别是dp[0][0]到dp[0][n-1]和dp[0][0]到dp[m-1][0]。可以看出就是dp数组的第0行和第0列。dp[0][j]的值应该是grid[0][0]到grid[0][j]的和（想要到达点$(0,j)$只能是从点$(0,0)$一路向右走），或者说dp[0][j] = dp[0][j-1]+grid[0][j]。同理有dp[i][0] = dp[i-1][0]+grid[i][0]
3. 状态转移方程：对于点$(i,j)$，达到该点之前的一个点只有两种可能：$(i-1, j)$和$(i,j-1)$。因此如果想要到该点的代价最小，那么只要其是从代价较小的前一个点到达即可。所以dp[i][j] = min(dp[i-1][j], dp[i][j-1])+grid[i][j]

最终实现代码如下：

```java
int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];
    for (int i = 1; i < m; i++) {
        dp[i][0] = dp[i-1][0] + grid[i][0];
    }
    for (int i = 1; i < n; i++) {
        dp[0][i] = dp[0][i - 1] + grid[0][i];
    }
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
    }
    return dp[m - 1][n - 1];
}
```

#### 零钱兑换

> 给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。
> 
> 示例1:
> 
>> 输入: coins = [1, 2, 5], amount = 11
>> 输出: 3 
>> 解释: 11 = 5 + 5 + 1
>
> 示例2:
> 
>> 输入: coins = [2], amount = 3
>> 输出: -1
>
> 说明: 你可以认为每种硬币的数量是无限的。

&#160; &#160; &#160; &#160; “目标点”比较不直观，是硬币的总面值达到amount；代价是硬币的个数，中间状态应该是在总面值达到amount之前的所有面值的最小硬币数目。

1. 确定状态：创建数组dp[amount+1]，其中dp[i]表示总面值为i的最少硬币数目，如果无法组成该数目，则为-1
2. 边界值：对于coins中所有的硬币面值c，都应该有dp[c]=1。
3. 状态转移方程：$dp[i]= \min_{c \in coins, dp[i-c]>0}{dp[i-c]}+1$，即对于总面值i，扫描coins中所有的单个面值c，找出代价最小的前一个状态。

```java
public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];

        Arrays.fill(dp, -1);
        dp[0] = 0;
        for (int i : coins) {
            if(i <= amount) {
                dp[i] = 1;
            }
        }

        for (int i = 1; i <= amount; i++) {
            if (dp[i] < 0) {
                int min = Integer.MAX_VALUE;
                for (int coin : coins) {
                    if(coin < i && dp[i-coin] > 0) {
                        min = Math.min(min, dp[i-coin]);
                    }
                }
                if(min < Integer.MAX_VALUE) {
                    dp[i] = min+1;
                }
                else dp[i] = -1;
            }
        }

        return dp[amount];
    }
```

#### 其他

* [746.使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)
* [931.下降路径最小和](https://leetcode-cn.com/problems/minimum-falling-path-sum/)
* [983. 最低票价](https://leetcode-cn.com/problems/minimum-cost-for-tickets/)
* [650. 只有两个键的键盘](https://leetcode-cn.com/problems/2-keys-keyboard/)
* [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)
* [1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/)
* [174. 地下城游戏](https://leetcode-cn.com/problems/dungeon-game/)
* [871. 最低加油次数](https://leetcode-cn.com/problems/minimum-number-of-refueling-stops/)

## 不同方法数

&#160; &#160; &#160; &#160; 这一类问题的描述通常是给定一个目标要求我们计算达到该目标的方法（路径）数有多少。创建状态转移方程的时候通常是找出所有前置状态，将其路径数相加，即dp[i]的含义一般是到达状态i的方法数。

### 不同路径

> [LeetCode.62 不同路径](https://leetcode-cn.com/problems/unique-paths/)一个机器人位于一个$m\times n$网格的左上角。机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
> 问总共有多少条不同的路径？
> 说明：m 和 n 的值均不超过 100。
>
>> 示例：
>> 输入: m = 3, n = 2
>> 输出: 3
>> 解释:
>> 从左上角开始，总共有 3 条路径可以到达右下角。
>> 1. 向右 -> 向右 -> 向下
>> 2. 向右 -> 向下 -> 向右
>> 3. 向下 -> 向右 -> 向右

1. 确定状态：dp[i][j]表示到达点(i,j)的路径数
2. 边界值：dp二维数组的第0列和第0行的值应该均为1
3. 状态转移方程：在到达点(i,j)之前，一定是在点(i-1,j)或者(i,j-1)，所以dp[i][j] = dp[i][j-1]+dp[i-1][j]

```java
public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1;
        }
        Arrays.fill(dp[0], 1);
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
```

### 其他

* [63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)
* [673. 最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)
* [1155. 掷骰子的N种方法](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/)

## 区间合并

&#160; &#160; &#160; &#160; 这类问题的描述一般为给定一列数，求解过程中要考虑的是数列中的某个数以及其左侧的子列和右侧的子列分别能得出的某些结论。一般有如下解题思路

```java
for(int l = 1; l<n; l++) {
   for(int i = 0; i<n-l; i++) {
       int j = i+l;
       for(int k = i; k<j; k++) {
           dp[i][j] = max(dp[i][j], dp[i][k] + result[k] + dp[k+1][j]);
       }
   }
}
 
return dp[0][n-1]
```

### 例题
* [96.不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)
* [375. 猜数字大小 II](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii/)
* [1130. 叶值的最小代价生成树](https://leetcode-cn.com/problems/minimum-cost-tree-from-leaf-values/)
* [1000. 合并石头的最低成本](https://leetcode-cn.com/problems/minimum-cost-to-merge-stones/)

## 字符串上的动态规划

&#160; &#160; &#160; &#160; 这一类题目一般都是给定两个字符串s1和s2，求一些值。一般来说要构建一个二维数组dp[][]，其中dp[i][j]表示在s1的第i个字符以及s2的第j个字符这样的状态的值。通常的解题过程为

```java
// i - indexing string s1
// j - indexing string s2
for (int i = 1; i <= n; ++i) {
   for (int j = 1; j <= m; ++j) {
       if (s1[i-1] == s2[j-1]) {
           dp[i][j] = /*code*/;
       } else {
           dp[i][j] = /*code*/;
       }
   }
}
```

### 最长公共子序列

> [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列。
> 
> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。
> 
> 若这两个字符串没有公共子序列，则返回 0。
> 示例：
>>  输入：text1 = "abcde", text2 = "ace" 
>> 输出：3  
>> 解释：最长公共子序列是 "ace"，它的长度为 3。

1. 确定状态：创建二维数组dp[len1][len2]，其中dp[i][j]表示到text1的第i个字符和text2的第j个字符时，最长的公共子序列的值。
2. 边界值：如果text1[0]==text2[0]，dp[0][0]= 1，否则dp[0][0]=0;
3. 状态转移方程：如果text1[i]==text2[j]，dp[i][j] = dp[i-1][j-1]+1，否则dp[i][j]=max(dp[i][j-1], dp[i-1][j]);

```java
public int longestCommonSubsequence(String text1, String text2) {
        int l1 = text1.length(), l2 = text2.length();
        int[][] dp = new int[l1][l2];

        if (text1.charAt(0) == text2.charAt(0)) dp[0][0] = 1;
        else dp[0][0] = 0;

        int max = 0;
        for (int i = 1; i < l1; i++) {
            if (text1.charAt(i) == text2.charAt(0)) {
                dp[i][0] = 1;
                max = 1;
            } else {
                dp[i][0] = dp[i-1][0];
            }
        }

        for (int i = 1; i < l2; i++) {
            if (text1.charAt(0) == text2.charAt(i)) {
                dp[0][i] = 1;
                max = 1;
            } else {
                dp[0][i] = dp[0][i-1];
            }
        }

        for (int i = 1; i < l1; i++) {
            for (int j = 1; j < l2; j++) {
                if (text1.charAt(i) == text2.charAt(j)) {
                    dp[i][j] = dp[i-1][j-1]+1;
                    max = Math.max(max, dp[i][j]);
                } else dp[i][j] = Math.max(dp[i][j-1], dp[i-1][j]);
            }
        }
        return max;
    }
```

### 其他

* [647. 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)
* [516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)
* [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

## 决策类问题

&#160; &#160; &#160; &#160;此模式的一般问题描述是针对给定情况决定是否使用当前状态。因此，问题需要您在当前状态下做出决定。一般的创建数组的思路是创建一个二维数组dp[n][k]，其中n为候选状态的数目，k为对状态能够采取的操作的数目。这样就可以保存对每一种状态采取所有不同措施的状态值。

### 例题

* [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)
* [714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
* [309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
* [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)
* [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

