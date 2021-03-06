# 动态规划
`"Those who cannot remember the past are condemned to repeat it."    -- Dynamic Programming`   

动态规划是一种带记忆的递归，通过将复杂问题分解成一个简单子问题集合，每个简单子问题只被解决一次并将它们的答案存在一个容器中。每个子问题的答案以某种方式被编号，使之能够被高效的查找，当相同的子问题下一次出现的时候，我们直接取它的答案直接用，而不是重新计算它。      

当使用动态规划解题时，首先要找到子问题（状态），然后明确`dp`数组的含义，明白`dp[i]`或`dp[i][j]`或`dp[i][j][k]`等等代表的是什么，最后寻找状态转移关系。   

## [经典DP问题](https://github.com/labuladong/fucking-algorithm/tree/master/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%B3%BB%E5%88%97)
### 1.KMP字符匹配问题
给定两个字符串：长度为`m`的模式串`pat`和长度为`n`的文本串`str`。在`str`中查找子串`pat`，如果存在，返回这个子串的起始索引，否则返回`-1`。  

这里的状态设置只与`pat`模式串有关，我们只需关注当前的匹配状态(匹配到第几(i)个字符，那么`dp[i]`的意思就是在当前匹配到`pat`中第i个字符时，遇到字符`str`中第j个字符后应转移到哪个状态。直至遇到最终态`dp[m]`或检测到`str`串已到达结尾。
```c++
int dp[1000];

// pat自己针对pat[1:]的匹配
void commonPrefix(string pat) {
    int m = pat.length();
    dp[0] = 0;
    int k = 0;
    for(int i = 1; i < m; ++i) {
        while(k > 0 && pat[k] != pat[i])
            k = dp[k - 1];
        if(pat[k] == pat[i])
            k = k + 1;
        dp[i] = k; 
    }
}

// str自己针对pat的匹配
int searchPat(string str, string pat) {
    int k = 0;
    int m = pat.length();
    for(int i = 0; i < str.length(); ++i) {
        while(k > 0 && pat[k] != str[i])
            k = dp[k - 1];
        if(pat[k] == str[i])
            k = k + 1;
        if(k == m)
            return i - m + 1;
    }
    return -1;
}
```

### 2.凑零钱问题
给你`k`种面值的硬币，面值分别为`[c1, c2 ... ck]`，每种硬币的数量无限，再给一个总金额 `amount`，问你最少需要几枚硬币凑出这个金额，如果不可能凑出，算法返回 `-1` 。
```c++
// coins 中是可选硬币面值，amount 是目标金额
int coinChange(int[] coins, int amount) {
    int dp[amount + 1];
    memset(dp, INT_MAX, sizeof(dp));
    // base-case
    dp[0] = 0;  // 当总金额为0时，所需最少金币数量也为0
    for(int i = 1; i <= amount; ++i) {
        for(auto coin : coins) {
            if(i - coin < 0)
                continue;
            dp[i] = min(dp[i], 1 + dp[i - coin]);
        }
    }
    return (dp[amount] == amount + 1) ? -1 : dp[amount];
}
```

### 3. 股票问题
[题目链接：121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)    
给定一个数组，里面的第`i`个元素代表这支股票在第`i`天的价格，在最多进行`k`笔交易(买入卖出算一次)的情况下，求出你可以获得的最大利润。   
[[题解：]](https://github.com/labuladong/fucking-algorithm/blob/master/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%B3%BB%E5%88%97/%E5%9B%A2%E7%81%AD%E8%82%A1%E7%A5%A8%E9%97%AE%E9%A2%98.md)
一组状态转移公式抗下所有！！！
```
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
              max(   选择 rest  ,             选择 sell     )

解释：今天我没有持有股票，有两种可能：
要么是我昨天就没有持有，然后今天选择 rest，所以我今天还是没有持有；
要么是我昨天持有股票，但是今天我 sell 了，所以我今天没有持有股票了。

dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
              max(   选择 rest  ,           选择 buy         )

解释：今天我持有着股票，有两种可能：
要么我昨天就持有着股票，然后今天选择 rest，所以我今天还持有着股票；
要么我昨天本没有持有，但今天我选择 buy，所以今天我就持有股票了。
```
初始状态(Base-Case)
```
dp[-1][k][0] = 0
解释：因为 i 是从 0 开始的，所以 i = -1 意味着还没有开始，这时候的利润当然是 0 。
dp[-1][k][1] = -infinity
解释：还没开始的时候，是不可能持有股票的，用负无穷表示这种不可能。
dp[i][0][0] = 0
解释：因为 k 是从 1 开始的，所以 k = 0 意味着根本不允许交易，这时候利润当然是 0 。
dp[i][0][1] = -infinity
解释：不允许交易的情况下，是不可能持有股票的，用负无穷表示这种不可能。
```

### 4. House Robber问题
[题目链接：198. House Robber](https://leetcode.com/problems/house-robber/)    
给定一个数组，里面的第`i`个元素代表这间房子里面存放的金额，但相邻两间房子不能在一晚同时被盗，否则会报警，求盗贼一晚上在不触发警报的情况下可以盗取的最大金额数。   
[[题解：]](https://leetcode.com/problems/house-robber/discuss/156523/From-good-to-great.-How-to-approach-most-of-DP-problems.)
```
This particular problem and most of others can be approached using the following sequence:
1. Find recursive relation  
2. Recursive (top-down)
3. Recursive + memo (top-down)
4. Iterative + memo (bottom-up)
5. Iterative + N variables (bottom-up)
```

### 5. 最长公共子序列（LCS）
[题目链接：1143. Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)   
给定两个字符串，求这两个字符串的最长公共子序列的长度：
```
输入: str1 = "abcde", str2 = "ace" 
输出: 3  
解释: 最长公共子序列是 "ace"，它的长度是 3
```
```c++
int longestCommonSubsequence(string text1, string text2) {
    int l1 = text1.length() + 1, l2 = text2.length() + 1;
    int i = 0, j = 0;
    int dp[l1 * l2];
    memset(dp, 0, sizeof(dp));
    for(i = 1; i < l1; ++i) {
        for(j = 1; j < l2; ++j) {
            if(text1[i-1] == text2[j-1])
                dp[j*l1+i] = dp[(j-1)*l1+(i-1)] + 1;
            else
                dp[j*l1+i] = max(dp[(j-1)*l1+i], dp[j*l1+i-1]);
        }
    }
    return dp[(l2-1)*l1+l1-1];
}
```

### 6. 博弈问题
[题目链接：877. Stone Game](https://leetcode.com/problems/stone-game/)     
给定一个数组代表有n堆石头，两个人每次只从当前数组的收尾两端进行选择，问最终是先手获得的石头多还是后手获得的石头多。   
[[题解：Optimal Strategy Game Pick from Ends of array Dynamic Programming]](https://www.youtube.com/watch?v=WxpIHvsu1RI)


### 7. 最长回文子序列
[题目链接：516. Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)   
解法与上面的最长公共子序列类似。  
```c++
int longestPalindromeSubseq(string s) {
    int n = s.length();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for(int i = 0; i < n; ++i) 
        dp[i][i] = 1;
    for(int i = n - 2; i >= 0; --i) {
        for(int j = i + 1; j < n; ++j) {
            if(s[i] == s[j])
                dp[i][j] = dp[i+1][j-1] + 2;
            else
                dp[i][j] = max(dp[i+1][j-1], max(dp[i][j-1], dp[i+1][j]));
        }
    }
    return dp[0][n-1];
}
```

### 8. 最短编辑距离
[题目链接：72. Edit Distance](https://leetcode.com/problems/edit-distance/)    
给定两个字符串`s1`和`s2`，求出将`s1`转换成`s2`的最小操作次数。（共包含三种操作，插入，删除，替换）    
[[题解：C++ O(n)-space DP]](https://leetcode.com/problems/edit-distance/discuss/25846/C%2B%2B-O(n)-space-DP)    
```c++
int minDistance(string word1, string word2) {
    int n = word1.length(), m = word2.length();
    // dp[i][j]代表word1[0..i-1]和word2[0..j-1]的两个字符串的最短编辑距离
    vector<vector<int>> dp(n+1, vector<int>(m+1, 0));
    for(int i = 0; i <= m; ++i)
        dp[0][i] = i;
    for(int j = 0; j <= n; ++j)
        dp[j][0] = j;
    for(int i = 1; i <= n; ++i) {
        for(int j = 1; j <= m; ++j) {
            if(word1[i-1] == word2[j-1])
                dp[i][j] = dp[i-1][j-1];
            else
                dp[i][j] = min(dp[i-1][j], min(dp[i-1][j-1], dp[i][j-1])) + 1; // (delete, replace, insert)
        }
    }
    return dp[n][m];
}
```
由于状态转移只与`dp[i-1][j], dp[i-1][j-1], dp[i][j-1]`这三个状态有关，所以在整个`n*m`的DP-Table中只有其前一行和当前行是有用的，因此我们可以将空间复杂度从`O(n*m)`压缩到`O(2*min(n, m))`，然后当前行需要维护的只有`dp[i][j-1]`所以我们找一个`tmp`变量来临时存放这个值就好，从而进一步将空间复杂度降低至`O(min(n, m))`。

### 9. 高楼扔鸡蛋问题
[题目链接：887. Super Egg Drop](https://leetcode.com/problems/super-egg-drop/)    
给你`K`个鸡蛋，`N`层的高楼，已知鸡蛋在高于某个楼层`F`的时候会摔碎，在`F`层以下抛出则不会摔碎，可以捡起来再次用于做实验，问你在最坏情况下最少做几次实验可以确定出`F`的具体数字来。       
[[题解：官方Solution]](https://leetcode.com/problems/super-egg-drop/solution/)   
```c++
// 直接DP-Table -- TLE(O(KN^2))
int superEggDrop(int K, int N) {
    vector<vector<int>> dp(K+1, vector<int>(N+1, 0));
    for(int i = 0; i <= N; ++i)
        dp[1][i] = i;
    for(int k = 2; k <= K; ++k) {
        for(int n = 1; n <= N; ++n) {
            dp[k][n] = INT_MAX;
            for(int i = 1; i <= n; ++i)
                dp[k][n] = min(dp[k][n], max(dp[k][n-i], dp[k-1][i-1]) + 1);
        }
    }
    return dp[K][N]; 
}
```
```c++
// 二分搜索优化 -- 
class Solution {
public:
    int superEggDrop(int K, int N) {
        vector<vector<int>> memo(K+1, vector<int>(N+1, -1));
        return dp(K, N, memo); 
    }
    
    int dp(int K, int N, vector<vector<int>>& memo) {
        if(K == 1)
            return N;
        if(N == 0)
            return 0;
        if(memo[K][N] != -1)
            return memo[K][N];
        int val = INT_MAX;
        int lo = 1, hi = N;
        # keep a gap of 2 X values to manually check later
        while(lo + 1 < hi) {
            int mid = (lo + hi) / 2;
            int t1 = dp(K-1, mid-1, memo);  // 递增随mid
            int t2 = dp(K, N-mid, memo);  // 递减随mid
            if(t1 < t2) {
                lo = mid;
            }
            else {
                hi = mid;
            }
        }
        for(int i = lo; i <= hi; ++i)
            val = min(val, max(dp(K, N-i, memo), dp(K-1, i-1, memo)) + 1);
        memo[K][N] = val;
        return val;
    }
};
```


Learn More: [Dynamic Programming](https://www.geeksforgeeks.org/dynamic-programming/)   

Practice More: [Top 50 Dynamic Programming Practice Problems](https://blog.usejournal.com/top-50-dynamic-programming-practice-problems-4208fed71aa3)




**习题：**  
* **[**[UVa10684:The jackpot](https://vjudge.net/problem/UVA-10684)**]** **[**[Solution(C++)][1]**]**
* **[**[UVa10755:Garbage Heap](https://vjudge.net/problem/UVA-10755)**]** **[**[Solution(C++)][1]**]**
* **[**[UVa983:Localized Summing for Blurring](https://vjudge.net/problem/UVA-983)**]** **[**[Solution(C++)][1]**]**
* **[**[UVa11951:Area](https://vjudge.net/problem/UVA-11951)**]** **[**[Solution(C++)][1]**]**
* **[**[UVa481:What Goes Up](https://vjudge.net/problem/UVA-481)**]** **[**[Solution(C++)][1]**]**
* **[**[UVa11368:Nested Dolls](https://vjudge.net/problem/UVA-11368)**]** **[**[Solution(C++)][1]**]**

[1]: https://github.com/Huixxi/Algorithm-with-Cplusplus/blob/master/Week01-%E5%9F%BA%E7%A1%80/UVa1585_Score.cpp
