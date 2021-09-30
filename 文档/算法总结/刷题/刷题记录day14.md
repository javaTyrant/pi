[toc]

#[886. 可能的二分法](https://leetcode-cn.com/problems/possible-bipartition/)(有趣的一题)

>   给定一组 N 人（编号为 1, 2, ..., N）， 我们想把每个人分进任意大小的两组。
>
>   每个人都可能不喜欢其他人，那么他们不应该属于同一组。
>
>   形式上，如果 dislikes[i] = [a, b]，表示不允许将编号为 a 和 b 的人归入同一组。
>
>   当可以用这种方法将所有人分进两组时，返回 true；否则返回 false。
>
>    
>
>   示例 1：
>
>   输入：N = 4, dislikes = [[1,2],[1,3],[2,4]]
>   输出：true
>   解释：group1 [1,4], group2 [2,3]
>   示例 2：
>
>   输入：N = 3, dislikes = [[1,2],[1,3],[2,3]]
>   输出：false
>   示例 3：
>
>   输入：N = 5, dislikes = [[1,2],[2,3],[3,4],[4,5],[1,5]]
>   输出：false
>
>
>   提示：
>
>   1 <= N <= 2000
>   0 <= dislikes.length <= 10000
>   dislikes[i].length == 2
>   1 <= dislikes[i][j] <= N
>   dislikes[i][0] < dislikes[i][1]
>   对于 dislikes[i] == dislikes[j] 不存在 i != j

```java
		public boolean possibleBipartition(int N, int[][] dislikes) {
        UnionFind uf = new UnionFind(N + 1);
        Map<Integer, Integer> map = new HashMap<>();
        Integer dislike = null; 
        for (int[] adjs : dislikes) {
            if ((dislike = map.get(adjs[0])) == null) {
                map.put(adjs[0], adjs[1]);
            } else {
                uf.union(adjs[1], dislike);
                if (uf.isConnected(adjs[0], dislike)) {
                    return false;
                }
            }
            if ((dislike = map.get(adjs[1])) == null) {
                map.put(adjs[1], adjs[0]);
            } else {
                uf.union(adjs[0], dislike);
                if (uf.isConnected(adjs[1], dislike)) {
                    return false;
                }
            }
        }
        return true;
    }
}

class UnionFind {

    private int[] roots = null;
    private int size = 0;

    UnionFind (int size) {
        this.size = size;
        roots = new int[size];
        for (int i = 0; i < size; i++) {
            roots[i] = i;
        }
    }

    public int findRoot(int i) {
        if (roots[i] == i) {
            return i;
        }
        return roots[i] = findRoot(roots[i]);
    }

    public boolean isConnected(int p, int q) {
        return findRoot(p) == findRoot(q);
    }

    public void union(int p, int q) {
        p = findRoot(p);
        q = findRoot(q);
        if (p == q) {
            return;
        } else if (p > q) {
            roots[p] = q;
        } else {
            roots[q] = p;
        }
        this.size--;
    }

    public int size() {
        return this.size;
    }
```

#[180. 连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/) 

>   编写一个 SQL 查询，查找所有至少连续出现三次的数字。
>
>   +----+-----+
>   | Id | Num |
>   +----+-----+
>   | 1  |  1  |
>   | 2  |  1  |
>   | 3  |  1  |
>   | 4  |  2  |
>   | 5  |  1  |
>   | 6  |  2  |
>   | 7  |  2  |
>   +----+-----+
>   例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。
>
>   +-----------------+
>   | ConsecutiveNums |
>   +-----------------+
>   | 1               |
>   +-----------------+

```java
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```

#[192. 统计词频](https://leetcode-cn.com/problems/word-frequency/) 

>   写一个 bash 脚本以统计一个文本文件 words.txt 中每个单词出现的频率。
>
>   为了简单起见，你可以假设：
>
>   words.txt只包括小写字母和 ' ' 。
>   每个单词只由小写字母组成。
>   单词间由一个或多个空格字符分隔。
>   示例:
>
>   假设 words.txt 内容如下：
>
>   the day is sunny the the
>   the sunny is is
>   你的脚本应当输出（以词频降序排列）：
>
>   the 4
>   is 3
>   sunny 2
>   day 1
>   说明:
>
>   不要担心词频相同的单词的排序问题，每个单词出现的频率都是唯一的。
>   你可以使用一行 Unix pipes 实现吗？

```java
# Read from the file words.txt and output the word frequency list to stdout.
awk '{for(i=1;i<=NF;i++){res[$i]+=1}}END{for(k in res){print k" "res[k]} }' words.txt | sort -nr -k2
```

#[268. 丢失的数字](https://leetcode-cn.com/problems/missing-number/) 

>   给定一个包含 [0, n] 中 n 个数的数组 nums ，找出 [0, n] 这个范围内没有出现在数组中的那个数。
>
>    
>
>   进阶：
>
>   你能否实现线性时间复杂度、仅使用额外常数空间的算法解决此问题?
>
>
>   示例 1：
>
>   输入：nums = [3,0,1]
>   输出：2
>   解释：n = 3，因为有 3 个数字，所以所有的数字都在范围 [0,3] 内。2 是丢失的数字，因为它没有出现在 nums 中。
>   示例 2：
>
>   输入：nums = [0,1]
>   输出：2
>   解释：n = 2，因为有 2 个数字，所以所有的数字都在范围 [0,2] 内。2 是丢失的数字，因为它没有出现在 nums 中。
>   示例 3：
>
>   输入：nums = [9,6,4,2,3,5,7,0,1]
>   输出：8
>   解释：n = 9，因为有 9 个数字，所以所有的数字都在范围 [0,9] 内。8 是丢失的数字，因为它没有出现在 nums 中。
>   示例 4：
>
>   输入：nums = [0]
>   输出：1
>   解释：n = 1，因为有 1 个数字，所以所有的数字都在范围 [0,1] 内。1 是丢失的数字，因为它没有出现在 nums 中。
>
>
>   提示：
>
>   n == nums.length
>   1 <= n <= 104
>   0 <= nums[i] <= n
>   nums 中的所有数字都 独一无二

```java
public int missingNumber(int[] nums) {
        int res = nums.length;
        for (int i = 0; i < nums.length; ++i){
            res ^= nums[i];
            res ^= i;
        }
        return res;
    }
```

#[450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/) 

>   

```java

```

#[503. 下一个更大元素 II](https://leetcode-cn.com/problems/next-greater-element-ii/) 

>   给定一个循环数组（最后一个元素的下一个元素是数组的第一个元素），输出每个元素的下一个更大元素。数字 x 的下一个更大的元素是按数组遍历顺序，这个数字之后的第一个比它更大的数，这意味着你应该循环地搜索它的下一个更大的数。如果不存在，则输出 -1。
>
>   示例 1:
>
>   输入: [1,2,1]
>   输出: [2,-1,2]
>   解释: 第一个 1 的下一个更大的数是 2；
>   数字 2 找不到下一个更大的数； 
>   第二个 1 的下一个最大的数需要循环搜索，结果也是 2。
>   注意: 输入数组的长度不会超过 10000。

```java
 public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int [] res = new int[n];
        Arrays.fill(res, -1);
        Stack <Integer> stack = new Stack<>();
        for (int i = 0; i < n*2; i++){
            int num = nums[i % n];
            while(!stack.isEmpty() && num > nums[stack.peek()]){
                res[stack.pop()] = num;
            }
            if(i < n) stack.add(i);
        }
        return res;
    }
```

#[58. 最后一个单词的长度](https://leetcode-cn.com/problems/length-of-last-word/) 

>   给定一个仅包含大小写字母和空格 ' ' 的字符串 s，返回其最后一个单词的长度。如果字符串从左向右滚动显示，那么最后一个单词就是最后出现的单词。
>
>   如果不存在最后一个单词，请返回 0 。
>
>   说明：一个单词是指仅由字母组成、不包含任何空格字符的 最大子字符串。
>
>    
>
>   示例:
>
>   输入: "Hello World"
>   输出: 5

```java
public int lengthOfLastWord(String s) {
        if(s == null) return 0;
        s = s.trim();
        int lastIndex = s.lastIndexOf(" ");
        String res = s.substring(lastIndex+1);
        if(res.equals("")) return 0;
        return res.length();
    }
```

#[60. 排列序列](https://leetcode-cn.com/problems/permutation-sequence/) 

>   给出集合 [1,2,3,...,n]，其所有元素共有 n! 种排列。
>
>   按大小顺序列出所有排列情况，并一一标记，当 n = 3 时, 所有排列如下：
>
>   "123"
>   "132"
>   "213"
>   "231"
>   "312"
>   "321"
>   给定 n 和 k，返回第 k 个排列。
>
>    
>
>   示例 1：
>
>   输入：n = 3, k = 3
>   输出："213"
>   示例 2：
>
>   输入：n = 4, k = 9
>   输出："2314"
>   示例 3：
>
>   输入：n = 3, k = 1
>   输出："123"
>
>
>   提示：
>
>   1 <= n <= 9
>   1 <= k <= n!

```java
 /**
        直接用回溯法做的话需要在回溯到第k个排列时终止就不会超时了, 但是效率依旧感人
        可以用数学的方法来解, 因为数字都是从1开始的连续自然数, 排列出现的次序可以推
        算出来, 对于n=4, k=15 找到k=15排列的过程:
        
        1 + 对2,3,4的全排列 (3!个)         
        2 + 对1,3,4的全排列 (3!个)         3, 1 + 对2,4的全排列(2!个)
        3 + 对1,2,4的全排列 (3!个)-------> 3, 2 + 对1,4的全排列(2!个)-------> 3, 2, 1 + 对4的全排列(1!个)-------> 3214
        4 + 对1,2,3的全排列 (3!个)         3, 4 + 对1,2的全排列(2!个)         3, 2, 4 + 对1的全排列(1!个)
        
        确定第一位:
            k = 14(从0开始计数)
            index = k / (n-1)! = 2, 说明第15个数的第一位是3 
            更新k
            k = k - index*(n-1)! = 2
        确定第二位:
            k = 2
            index = k / (n-2)! = 1, 说明第15个数的第二位是2
            更新k
            k = k - index*(n-2)! = 0
        确定第三位:
            k = 0
            index = k / (n-3)! = 0, 说明第15个数的第三位是1
            更新k
            k = k - index*(n-3)! = 0
        确定第四位:
            k = 0
            index = k / (n-4)! = 0, 说明第15个数的第四位是4
        最终确定n=4时第15个数为3214 
        **/
        
        StringBuilder sb = new StringBuilder();
        // 候选数字
        List<Integer> candidates = new ArrayList<>();
        // 分母的阶乘数
        int[] factorials = new int[n+1];
        factorials[0] = 1;
        int fact = 1;
        for(int i = 1; i <= n; ++i) {
            candidates.add(i);
            fact *= i;
            factorials[i] = fact;
        }
        k -= 1;
        for(int i = n-1; i >= 0; --i) {
            // 计算候选数字的index
            int index = k / factorials[i];
            sb.append(candidates.remove(index));
            k -= index*factorials[i];
        }
        return sb.toString();
    }
```

#[27. 移除元素](https://leetcode-cn.com/problems/remove-element/) 

>   给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。
>
>   不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。
>
>   元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
>
>    
>
>   示例 1:
>
>   给定 nums = [3,2,2,3], val = 3,
>
>   函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。
>
>   你不需要考虑数组中超出新长度后面的元素。
>   示例 2:
>
>   给定 nums = [0,1,2,2,3,0,4,2], val = 2,
>
>   函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。
>
>   注意这五个元素可为任意顺序。
>
>   你不需要考虑数组中超出新长度后面的元素。
>
>
>   说明:
>
>   为什么返回数值是整数，但输出的答案是数组呢?
>
>   请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。
>
>   你可以想象内部操作如下:
>
>   // nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
>   int len = removeElement(nums, val);
>
>   // 在函数里修改输入数组对于调用者是可见的。
>   // 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
>   for (int i = 0; i < len; i++) {
>       print(nums[i]);
>   }

```java
public int removeElement(int[] nums, int val) {
     int res = 0;
        for(int i=0; i<nums.length;i++){
            if(nums[i]!=val)
                nums[res++] = nums[i];
        }
        return res;
    }
```

#[395. 至少有K个重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-with-at-least-k-repeating-characters/) 

>   找到给定字符串（由小写字符组成）中的最长子串 T ， 要求 T 中的每一字符出现次数都不少于 k 。输出 T 的长度。
>
>   示例 1:
>
>   输入:
>   s = "aaabb", k = 3
>
>   输出:
>   3
>
>   最长子串为 "aaa" ，其中 'a' 重复了 3 次。
>   示例 2:
>
>   输入:
>   s = "ababbc", k = 2
>
>   输出:
>   5
>
>   最长子串为 "ababb" ，其中 'a' 重复了 2 次， 'b' 重复了 3 次。
>

```java
 public int longestSubstring(String s, int k) {
          if (k < 2) return s.length();
          return process(s.toCharArray(), k, 0, s.length() - 1);
    }
    private int process(char[] s, int k, int lo, int hi) {
    if (hi - lo + 1 < k) return 0;

    int[] cnts = new int[26];
    for (int i = lo; i <= hi; ++i) cnts[s[i]-'a']++;

    while (hi - lo + 1 >= k && cnts[s[lo]-'a'] < k) lo++;
    while (hi - lo + 1 >= k && cnts[s[hi]-'a'] < k) hi--;
    if (hi - lo + 1 < k) return 0;

    for (int i = lo; i <= hi; ++i)
        if (cnts[s[i]-'a'] < k) return Math.max(process(s, k, lo, i - 1), process(s, k, i + 1, hi));

    return hi - lo + 1;
    }
```

#[516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/) 

>   给定一个字符串 s ，找到其中最长的回文子序列，并返回该序列的长度。可以假设 s 的最大长度为 1000 。
>
>    
>
>   示例 1:
>   输入:
>
>   "bbbab"
>   输出:
>
>   4
>   一个可能的最长回文子序列为 "bbbb"。
>
>   示例 2:
>   输入:
>
>   "cbbd"
>   输出:
>
>   2
>   一个可能的最长回文子序列为 "bb"。
>
>    
>
>   提示：
>
>   1 <= s.length <= 1000
>   s 只包含小写英文字母

```java
public int longestPalindromeSubseq(String s) {
         //dp[i,j] i到 j 表示 i j 范围内的最长子序列的长度
        //最短为1
        char[] p = s.toCharArray();
        int[][]dp = new int[p.length+1][p.length+1];
        int n = p.length;
        for(int i=1;i<=n;++i)dp[i][i] = 1; 


        for(int i=n-1; i>0;--i) {
            for(int j=i+1;j<=n;++j) {
                if(p[i-1]==p[j-1] ) {
                    dp[i][j] = dp[i+1][j-1]+2;
                }else {
                    dp[i][j] = Math.max(dp[i+1][j],dp[i][j-1]);
                }
                //else

            }
        }
        return dp[1][n];
    }
```

#[131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/) 

>   给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。
>
>   返回 s 所有可能的分割方案。
>
>   示例:
>
>   输入: "aab"
>   输出:
>   [
>     ["aa","b"],
>     ["a","a","b"]
>   ]

```java
public List<List<String>> partition(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        for (int len = 1; len <= n; len++) {
            int j;
            for (int i = 0; (j = i + len - 1) < n; i++)
                dp[i][j] = len == 1 || s.charAt(i) == s.charAt(j) && (len == 2 || dp[i + 1][j - 1]);
        }
        List<List<String>> ans = new ArrayList<>();
        dfs(s, dp, 0, new ArrayList<>(), ans);
        return ans;
    }

    private void dfs(String s, boolean[][] dp, int begin, List<String> buffer, List<List<String>> ans) {
        if (begin == s.length()) {
            ans.add(new ArrayList<>(buffer));
            return;
        }
        int end;
        for (int len = 1; (end = begin + len - 1) < s.length(); len++) {
            if (dp[begin][end]) {
                buffer.add(s.substring(begin, end + 1));
                dfs(s, dp, begin + len, buffer, ans);
                buffer.remove(buffer.size() - 1);
            }
        }
    }
```

#[210. 课程表 II](https://leetcode-cn.com/problems/course-schedule-ii/) 

>   现在你总共有 n 门课需要选，记为 0 到 n-1。
>
>   在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: [0,1]
>
>   给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。
>
>   可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。
>
>   示例 1:
>
>   输入: 2, [[1,0]] 
>   输出: [0,1]
>   解释: 总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1] 。
>   示例 2:
>
>   输入: 4, [[1,0],[2,0],[3,1],[3,2]]
>   输出: [0,1,2,3] or [0,2,1,3]
>   解释: 总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
>        因此，一个正确的课程顺序是 [0,1,2,3] 。另一个正确的排序是 [0,2,1,3] 。
>   说明:
>
>   输入的先决条件是由边缘列表表示的图形，而不是邻接矩阵。详情请参见图的表示法。
>   你可以假定输入的先决条件中没有重复的边。
>   提示:
>
>   这个问题相当于查找一个循环是否存在于有向图中。如果存在循环，则不存在拓扑排序，因此不可能选取所有课程进行学习。
>   通过 DFS 进行拓扑排序 - 一个关于Coursera的精彩视频教程（21分钟），介绍拓扑排序的基本概念。
>   拓扑排序也可以通过 BFS 完成。

```java
public int[] findOrder(int n, int[][] brr) {
        int arr[] = new int[n];
        Map<Integer, Set<Integer>> map = new HashMap<>();
        Set<Integer> set = new HashSet<>();
        for (int i = 0; i < brr.length; i++) {
            arr[brr[i][0]]++;
            set = map.get(brr[i][1]);
            if (set == null) {
                set = new HashSet<>();
                set.add(brr[i][0]);
                map.put(brr[i][1], set);
            }else {
                set.add(brr[i][0]);
            }
        }
        List<Integer> list = new ArrayList<>();
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == 0) {
                queue.offer(i);
            }
        }
        while (!queue.isEmpty()) {
            Integer now = queue.poll();
            list.add(now);
            set = map.get(now);
            if(set!=null){
                for (Integer nn : set) {
                    if(--arr[nn]==0)
                        queue.offer(nn);
                }

            }
        }
          if(list.size()!=n)
              return new  int[0];
        int crr[] = new int[list.size()];
        for (int i = 0; i < crr.length; i++) {
            crr[i] = list.get(i);
        }
        return crr;
    }
```

#[37. 解数独](https://leetcode-cn.com/problems/sudoku-solver/) 

>   编写一个程序，通过填充空格来解决数独问题。
>
>   一个数独的解法需遵循如下规则：
>
>   数字 1-9 在每一行只能出现一次。
>   数字 1-9 在每一列只能出现一次。
>   数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。
>   空白格用 '.' 表示。
>
>   ![img](http://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Sudoku-by-L2G-20050714.svg/250px-Sudoku-by-L2G-20050714.svg.png)
>
>   ![img](http://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Sudoku-by-L2G-20050714_solution.svg/250px-Sudoku-by-L2G-20050714_solution.svg.png)

```java
 // box size
    int n = 3;
    // row size
    int N = n * n;

    int[][] rows = new int[N][N + 1];
    int[][] columns = new int[N][N + 1];
    int[][] boxes = new int[N][N + 1];

    char[][] board;

    boolean sudokuSolved = false;

    public boolean couldPlace(int d, int row, int col) {
    /*
    Check if one could place a number d in (row, col) cell
    */
        int idx = (row / n) * n + col / n;
        return rows[row][d] + columns[col][d] + boxes[idx][d] == 0;
    }

    public void placeNumber(int d, int row, int col) {
    /*
    Place a number d in (row, col) cell
    */
        int idx = (row / n) * n + col / n;

        rows[row][d]++;
        columns[col][d]++;
        boxes[idx][d]++;
        board[row][col] = (char) (d + '0');
    }

    public void removeNumber(int d, int row, int col) {
    /*
    Remove a number which didn't lead to a solution
    */
        int idx = (row / n) * n + col / n;
        rows[row][d]--;
        columns[col][d]--;
        boxes[idx][d]--;
        board[row][col] = '.';
    }

    public void placeNextNumbers(int row, int col) {
    /*
    Call backtrack function in recursion
    to continue to place numbers
    till the moment we have a solution
    */
        // if we're in the last cell
        // that means we have the solution
        if ((col == N - 1) && (row == N - 1)) {
            sudokuSolved = true;
        }
        // if not yet
        else {
            // if we're in the end of the row
            // go to the next row
            if (col == N - 1) backtrack(row + 1, 0);
                // go to the next column
            else backtrack(row, col + 1);
        }
    }

    public void backtrack(int row, int col) {
    /*
    Backtracking
    */
        // if the cell is empty
        if (board[row][col] == '.') {
            // iterate over all numbers from 1 to 9
            for (int d = 1; d < 10; d++) {
                if (couldPlace(d, row, col)) {
                    placeNumber(d, row, col);
                    placeNextNumbers(row, col);
                    // if sudoku is solved, there is no need to backtrack
                    // since the single unique solution is promised
                    if (!sudokuSolved) removeNumber(d, row, col);
                }
            }
        } else placeNextNumbers(row, col);
    }

    public void solveSudoku(char[][] board) {
        this.board = board;

        // init rows, columns and boxes
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                char num = board[i][j];
                if (num != '.') {
                    int d = Character.getNumericValue(num);
                    placeNumber(d, i, j);
                }
            }
        }
        backtrack(0, 0);
    }
```

#[380. 常数时间插入、删除和获取随机](https://leetcode-cn.com/problems/insert-delete-getrandom-o1/) 

>   
>   设计一个支持在*平均* 时间复杂度 **O(1)** 下，执行以下操作的数据结构。
>
>   1. `insert(val)`：当元素 val 不存在时，向集合中插入该项。
>   2. `remove(val)`：元素 val 存在时，从集合中移除该项。
>   3. `getRandom`：随机返回现有集合中的一项。每个元素应该有**相同的概率**被返回。
>
>   **示例 :**
>
>   ```sql
>   // 初始化一个空的集合。
>   RandomizedSet randomSet = new RandomizedSet();
>   
>   // 向集合中插入 1 。返回 true 表示 1 被成功地插入。
>   randomSet.insert(1);
>   
>   // 返回 false ，表示集合中不存在 2 。
>   randomSet.remove(2);
>   
>   // 向集合中插入 2 。返回 true 。集合现在包含 [1,2] 。
>   randomSet.insert(2);
>   
>   // getRandom 应随机返回 1 或 2 。
>   randomSet.getRandom();
>   
>   // 从集合中移除 1 ，返回 true 。集合现在包含 [2] 。
>   randomSet.remove(1);
>   
>   // 2 已在集合中，所以返回 false 。
>   randomSet.insert(2);
>   
>   // 由于 2 是集合中唯一的数字，getRandom 总是返回 2 。
>   randomSet.getRandom();
>   ```

```java
Map<Integer, Integer> dict;
  List<Integer> list;
  Random rand = new Random();

  /** Initialize your data structure here. */
  public RandomizedSet() {
    dict = new HashMap();
    list = new ArrayList();
  }

  /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
  public boolean insert(int val) {
    if (dict.containsKey(val)) return false;

    dict.put(val, list.size());
    list.add(list.size(), val);
    return true;
  }

  /** Removes a value from the set. Returns true if the set contained the specified element. */
  public boolean remove(int val) {
    if (! dict.containsKey(val)) return false;

    // move the last element to the place idx of the element to delete
    int lastElement = list.get(list.size() - 1);
    int idx = dict.get(val);
    list.set(idx, lastElement);
    dict.put(lastElement, idx);
    // delete the last element
    list.remove(list.size() - 1);
    dict.remove(val);
    return true;
  }

  /** Get a random element from the set. */
  public int getRandom() {
    return list.get(rand.nextInt(list.size()));
  }
```

#[剑指 Offer 52 - 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

>   输入两个链表，找出它们的第一个公共节点。
>
>   如下面的两个链表：
>
>   
>
>   在节点 c1 开始相交。
>
>    
>
>   示例 1：
>
>   
>
>   输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
>   输出：Reference of the node with value = 8
>   输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
>
>
>   示例 2：
>
>   
>
>   输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
>   输出：Reference of the node with value = 2
>   输入解释：相交节点的值为 2 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
>
>
>   示例 3：
>
>   
>
>   输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
>   输出：null
>   输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
>   解释：这两个链表不相交，因此返回 null。

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode p1 = headA,p2 = headB;
        while(p1 != p2){
            p1 = (p1 == null) ? headB : p1.next;
            p2 = (p2 == null) ? headA : p2.next;
        }
        return p1;
    }
```

#[面试题 02.03 - 删除中间节点](https://leetcode-cn.com/problems/delete-middle-node-lcci/) 

>   
>   实现一种算法，删除单向链表中间的某个节点（即不是第一个或最后一个节点），假定你只能访问该节点。
>
>    ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_1.png)
>
>   **示例：**
>
>   ```sql
>   输入：单向链表a->b->c->d->e->f中的节点c
>   结果：不返回任何数据，但该链表变为a->b->d->e->f
>   ```

```java
 public void deleteNode(ListNode node) {
        node.val=node.next.val;
        node.next=node.next.next;  
    }
```

#[116. 填充每个节点的下一个右侧节](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/) 

>   给定一个完美二叉树，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：
>
>   struct Node {
>     int val;
>     Node *left;
>     Node *right;
>     Node *next;
>   }
>   填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。
>
>   初始状态下，所有 next 指针都被设置为 NULL。
>
>    
>
>   示例：
>
>   
>
>   输入：{"$id":"1","left":{"$id":"2","left":{"$id":"3","left":null,"next":null,"right":null,"val":4},"next":null,"right":{"$id":"4","left":null,"next":null,"right":null,"val":5},"val":2},"next":null,"right":{"$id":"5","left":{"$id":"6","left":null,"next":null,"right":null,"val":6},"next":null,"right":{"$id":"7","left":null,"next":null,"right":null,"val":7},"val":3},"val":1}
>
>   输出：{"$id":"1","left":{"$id":"2","left":{"$id":"3","left":null,"next":{"$id":"4","left":null,"next":{"$id":"5","left":null,"next":{"$id":"6","left":null,"next":null,"right":null,"val":7},"right":null,"val":6},"right":null,"val":5},"right":null,"val":4},"next":{"$id":"7","left":{"$ref":"5"},"next":null,"right":{"$ref":"6"},"val":3},"right":{"$ref":"4"},"val":2},"next":null,"right":{"$ref":"7"},"val":1}
>
>   解释：给定二叉树如图 A 所示，你的函数应该填充它的每个 next 指针，以指向其下一个右侧节点，如图 B 所示。
>

```java
 public Node connect(Node root) {
        if(root == null || root.left == null)
            return root;
        root.left.next = root.right;
        if(root.next != null)
            root.right.next = root.next.left;
        connect(root.left);
        connect(root.right);
        return root;
    }
```

#[剑指 Offer 32 - III - 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/) 

>   从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。
>
>    
>
>   例如:
>   给定二叉树: [3,9,20,null,null,15,7],
>
>       3
>      / \
>     9  20
>       /  \
>      15   7
>   返回：
>
>   [3,9,20,15,7]

```java
public int[] levelOrder(TreeNode root) {
        if(root == null) return new int[0];
        List<Integer> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
           TreeNode current =  queue.poll();
           res.add(current.val);
           if(current.left != null) queue.add(current.left);
           if(current.right != null) queue.add(current.right);
        }
        int[] r = new int[res.size()];
        for(int i = 0;i<res.size();i++){
            r[i] = res.get(i);
        }
        return r;
    }
```

#[剑指 Offer 39 - 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/) 

>   
>   数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。
>
>    
>
>   你可以假设数组是非空的，并且给定的数组总是存在多数元素。
>
>    
>
>   **示例 1:**
>
>   ```sql
>   输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
>   输出: 2
>   ```
>
>    
>
>   **限制：**
>
>   ```
>   1 <= 数组长度 <= 50000
>   ```

```java
  public int majorityElement(int[] nums) {
       int majority = 0;
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            if (count == 0) {
                majority = nums[i];
                count++;
            } else if (nums[i] == majority) {
                count++;
            } else {
                count--;
            }
        }
        return majority;
    }
```

