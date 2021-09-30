## [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

## [最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

## [ 最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

## [不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

## [正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

## [ 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

## [不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

## [ 戳气球](https://leetcode-cn.com/problems/burst-balloons/)

## [最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

## [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

## [ 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

## [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

## [ 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

## [ 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

## [ 移除盒子](https://leetcode-cn.com/problems/remove-boxes/)

## [ 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

## [ 统计重复个数](https://leetcode-cn.com/problems/count-the-repetitions/)

## [ 最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/)

## [最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

## [划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)

> ```sql
> [4, 3, 2, 3, 5, 2, 1]
> k = 4
> ```
>
> 执行路径
>
> sum = 20 k = 4说明每个子集的和必须==5
>
> target = 5
>
> 排序[1,2,2,3,3,4,5]
>
> 如果最后一个数>target直接返回false.不可能有满足的条件
>
> row = 6
>
> 从最后一个数字找,因为最后一个数字=5满足,所以k减1,row-1
>
> 还需要找到3个子集
>
> search(new int[3],5,nums,5)
>
> row < 0吗 否
>
> v = 4