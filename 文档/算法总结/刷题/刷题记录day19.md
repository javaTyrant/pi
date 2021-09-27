[toc]

#[856. 括号的分数](https://leetcode-cn.com/problems/score-of-parentheses/) 

>  给定一个平衡括号字符串 S，按下述规则计算该字符串的分数：
>
>  () 得 1 分。
>  AB 得 A + B 分，其中 A 和 B 是平衡括号字符串。
>  (A) 得 2 * A 分，其中 A 是平衡括号字符串。
>
>
>  示例 1：
>
>  输入： "()"
>  输出： 1
>  示例 2：
>
>  输入： "(())"
>  输出： 2
>  示例 3：
>
>  输入： "()()"
>  输出： 2
>  示例 4：
>
>  输入： "(()(()))"
>  输出： 6

```java
 public int scoreOfParentheses(String S) {
        int length=S.length();
        int sum=0;
        int n=0;
        for(int i=0;i<length;i++)
        {
            if(S.charAt(i)=='(')
            {
                if(n==0)
                {
                    n=1;
                }
                else
                {
                    n=n<<1;
                }
            }
            else if(S.charAt(i)==')')
            {
                 if(S.charAt(i-1)=='(')
                {
                     sum+=n;
                }
                n=n>>1;
                
            }
        }
        return sum;
    }
```



#[223. 矩形面积](https://leetcode-cn.com/problems/rectangle-area/) 

>  在二维平面上计算出两个由直线构成的矩形重叠后形成的总面积。
>
>  每个矩形由其左下顶点和右上顶点坐标表示，如图所示。
>
>  ![Rectangle Area](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rectangle_area.png)
>
>  示例:
>
>  输入: -3, 0, 3, 4, 0, -1, 9, 2
>  输出: 45
>  说明: 假设矩形面积不会超出 int 的范围。

```java
   public int computeArea(int A, int B, int C, int D, int E, int F, int G, int H) {
        int s = 0;
        if (E >= C || G <= A || H <= B || F >= D) {
            s = 0;
        } else {
            int x1 = Math.max(A, E);
            int x2 = Math.min(C, G);
        
            int y1 = Math.max(B, F);
            int y2 = Math.min(D, H);
        
            s = (x2 - x1) * (y2 - y1);
        }
        
        return (C-A) * (D-B) + (G-E) * (H-F) - s;
    }
```



#[454. 四数相加 II](https://leetcode-cn.com/problems/4sum-ii/) 

>  给定四个包含整数的数组列表 A , B , C , D ,计算有多少个元组 (i, j, k, l) ，使得 A[i] + B[j] + C[k] + D[l] = 0。
>
>  为了使问题简单化，所有的 A, B, C, D 具有相同的长度 N，且 0 ≤ N ≤ 500 。所有整数的范围在 -228 到 228 - 1 之间，最终结果不会超过 231 - 1 。
>
>  例如:
>
>  输入:
>  A = [ 1, 2]
>  B = [-2,-1]
>  C = [-1, 2]
>  D = [ 0, 2]
>
>  输出:
>  2
>
>  解释:
>  两个元组如下:
>  1. (0, 0, 0, 1) -> A[0] + B[0] + C[0] + D[1] = 1 + (-2) + (-1) + 2 = 0
>  2. (1, 1, 0, 0) -> A[1] + B[1] + C[0] + D[0] = 2 + (-1) + (-1) + 0 = 0
>

```java
  public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Map<Integer, Integer> mapAB = new HashMap<>();
        int res = 0;

        for (int a : A) {
            for (int b : B) {
                int key = a + b;
                if (mapAB.containsKey(key))
                    mapAB.put(key, mapAB.get(key) + 1);
                else mapAB.put(key, 1);
            }
        }

        for (int c : C) {
            for (int d : D) {
                int key = c + d;
                if (mapAB.containsKey(0 - key)) {
                    res += mapAB.get(0 - key);
                }
            }
        }
        return res;
    }
```



#[596. 超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/) 

>  有一个courses 表 ，有: student (学生) 和 class (课程)。
>
>  请列出所有超过或等于5名学生的课。
>
>  例如，表：
>
>  +---------+------------+
>  | student | class      |
>  +---------+------------+
>  | A       | Math       |
>  | B       | English    |
>  | C       | Math       |
>  | D       | Biology    |
>  | E       | Math       |
>  | F       | Computer   |
>  | G       | Math       |
>  | H       | Math       |
>  | I       | Math       |
>  +---------+------------+
>  应该输出:
>
>  +---------+
>  | class   |
>  +---------+
>  | Math    |
>  +---------+
>
>  
>

```java
# Write your MySQL query statement below
select class from courses group by class having count(distinct student) >= 5
```



#[703. 数据流中的第 K 大元素](https://leetcode-cn.com/problems/kth-largest-element-in-a-stream/) 

>  
>  设计一个找到数据流中第 `k` 大元素的类（class）。注意是排序后的第 `k` 大元素，不是第 `k` 个不同的元素。
>
>  请实现 `KthLargest` 类：
>
>  - `KthLargest(int k, int[] nums)` 使用整数 `k` 和整数流 `nums` 初始化对象。
>  - `int add(int val)` 返回当前数据流中第 `k` 大的元素。
>
>   
>
>  **示例：**
>
>  ```
>  输入：
>  ["KthLargest", "add", "add", "add", "add", "add"]
>  [[3, [4, 5, 8, 2]], [3], [5], [10], [9], [4]]
>  输出：
>  [null, 4, 5, 5, 8, 8]
>  
>  解释：
>  KthLargest kthLargest = new KthLargest(3, [4, 5, 8, 2]);
>  kthLargest.add(3);   // return 4
>  kthLargest.add(5);   // return 5
>  kthLargest.add(10);  // return 5
>  kthLargest.add(9);   // return 8
>  kthLargest.add(4);   // return 8
>  ```
>
>   
>
>  **提示：**
>
>  - `1 <= k <= 104`
>  - `0 <= nums.length <= 104`
>  - `-104 <= nums[i] <= 104`
>  - `-104 <= val <= 104`
>  - 最多调用 `add` 方法 `104` 次
>  - 题目数据保证，在查找第 `k` 大元素时，数组中至少有 `k` 个元素

```java
final PriorityQueue<Integer> pq;
    final int k;
    
    public KthLargest(int k, int[] nums) {
        this.k = k;
        pq = new PriorityQueue<>(k);
 
        for(int i : nums){//对传进来的int数组遍历
            add(i);
        }
    }
    
    public int add(int val) {
        if(pq.size() < k)//如果队列中的数量少于K，直接添加入优先队列，优先队列会自动维持小顶堆
            pq.offer(val);
        else{
            if(pq.peek() < val){//否则队列中的数量大于或者等于K，优先队列中的最小数字小于新的数据，优先队列中的顶堆要被移除，并且添加入新的数据进优先队列
                pq.poll();
                pq.offer(val);
 
            }        
 
        }
        return pq.peek();//返回当前第K大的数
 
    }
```



#[382. 链表随机节点](https://leetcode-cn.com/problems/linked-list-random-node/) 

>  给定一个单链表，随机选择链表的一个节点，并返回相应的节点值。保证每个节点被选的概率一样。
>
>  进阶:
>  如果链表十分大且长度未知，如何解决这个问题？你能否使用常数级空间复杂度实现？
>
>  示例:
>
>  // 初始化一个单链表 [1,2,3].
>  ListNode head = new ListNode(1);
>  head.next = new ListNode(2);
>  head.next.next = new ListNode(3);
>  Solution solution = new Solution(head);
>
>  // getRandom()方法应随机返回1,2,3中的一个，保证每个元素被返回的概率相等。
>  solution.getRandom();

```java
 private ListNode head;
    public Solution(ListNode head) {
       this.head = head;
    }
 
    public int getRandom() {
        int res = head.val;
        ListNode no = head.next;
        int i = 2;
        Random random = new Random();
        while(no!=null){
            if(random.nextInt(i) == 0){
                res = no.val;
            }
            i++;
            no = no.next;
        }
        return res;

    }
```



#[494. 目标和](https://leetcode-cn.com/problems/target-sum/) 

>  给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。
>
>  返回可以使最终数组和为目标数 S 的所有添加符号的方法数。
>
>   
>
>  示例：
>
>  输入：nums: [1, 1, 1, 1, 1], S: 3
>  输出：5
>  解释：
>
>  -1+1+1+1+1 = 3
>  +1-1+1+1+1 = 3
>  +1+1-1+1+1 = 3
>  +1+1+1-1+1 = 3
>  +1+1+1+1-1 = 3
>
>  一共有5种方法让最终目标和为3。
>
>
>  提示：
>
>  数组非空，且长度不会超过 20 。
>  初始的数组的和不会超过 1000 。
>  保证返回的最终结果能被 32 位整数存下。

```java
 public int findTargetSumWays(int[] nums, int S) {
          int[][] dp = new int[nums.length][2001];
        dp[0][nums[0] + 1000] = 1;
        dp[0][-nums[0] + 1000] += 1;
        for (int i = 1; i < nums.length; i++) {
            for (int sum = -1000; sum <= 1000; sum++) {
                if (dp[i - 1][sum + 1000] > 0) {
                    dp[i][sum + nums[i] + 1000] += dp[i - 1][sum + 1000];
                    dp[i][sum - nums[i] + 1000] += dp[i - 1][sum + 1000];
                }
            }
        }
        return S > 1000 ? 0 : dp[nums.length - 1][S + 1000];
    }
```



#[834. 树中距离之和](https://leetcode-cn.com/problems/sum-of-distances-in-tree/) 

>  给定一个无向、连通的树。树中有 N 个标记为 0...N-1 的节点以及 N-1 条边 。
>
>  第 i 条边连接节点 edges[i][0] 和 edges[i][1] 。
>
>  返回一个表示节点 i 与其他所有节点距离之和的列表 ans。
>
>  示例 1:
>
>  输入: N = 6, edges = [[0,1],[0,2],[2,3],[2,4],[2,5]]
>  输出: [8,12,6,10,10,10]
>  解释: 
>  如下为给定的树的示意图：
>    0
>   / \
>  1   2
>     /|\
>    3 4 5
>
>  我们可以计算出 dist(0,1) + dist(0,2) + dist(0,3) + dist(0,4) + dist(0,5) 
>  也就是 1 + 1 + 2 + 2 + 2 = 8。 因此，answer[0] = 8，以此类推。
>  说明: 1 <= N <= 10000

```java
int[] ans;
    int[] sz;
    int[] dp;
    List<List<Integer>> graph;

    public int[] sumOfDistancesInTree(int N, int[][] edges) {
        ans = new int[N];
        sz = new int[N];
        dp = new int[N];
        graph = new ArrayList<List<Integer>>();
        for (int i = 0; i < N; ++i) {
            graph.add(new ArrayList<Integer>());
        }
        for (int[] edge: edges) {
            int u = edge[0], v = edge[1];
            graph.get(u).add(v);
            graph.get(v).add(u);
        }
        dfs(0, -1);
        dfs2(0, -1);
        return ans;
    }

    public void dfs(int u, int f) {
        sz[u] = 1;
        dp[u] = 0;
        for (int v: graph.get(u)) {
            if (v == f) {
                continue;
            }
            dfs(v, u);
            dp[u] += dp[v] + sz[v];
            sz[u] += sz[v];
        }
    }

    public void dfs2(int u, int f) {
        ans[u] = dp[u];
        for (int v: graph.get(u)) {
            if (v == f) {
                continue;
            }
            int pu = dp[u], pv = dp[v];
            int su = sz[u], sv = sz[v];

            dp[u] -= dp[v] + sz[v];
            sz[u] -= sz[v];
            dp[v] += dp[u] + sz[u];
            sz[v] += sz[u];

            dfs2(v, u);

            dp[u] = pu;
            dp[v] = pv;
            sz[u] = su;
            sz[v] = sv;
        }
    }
```



#[637. 二叉树的层平均值](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/) 

>  给定一个非空二叉树, 返回一个由每层节点平均值组成的数组。
>
>   
>
>  示例 1：
>
>  输入：
>      3
>     / \
>    9  20
>      /  \
>     15   7
>  输出：[3, 14.5, 11]
>  解释：
>  第 0 层的平均值是 3 ,  第1层是 14.5 , 第2层是 11 。因此返回 [3, 14.5, 11] 。
>
>
>  提示：
>
>  节点值的范围在32位有符号整数范围内。
>

```java
 public List<Double> averageOfLevels(TreeNode root) {
        List<double[]> sumCountList = new ArrayList<>();
        levelOrderBottom(root, 0, sumCountList);
        List<Double> res = new ArrayList<>();
        for (double[] sumCount : sumCountList) {
            res.add(sumCount[0] * 1.0d / sumCount[1]);
        }
        return res;
    }

    private void levelOrderBottom(TreeNode root, int level, List<double[]> res) {
        if (root == null) return;
        if (res.size() <= level) res.add(new double[2]);
        double[] sumCount = res.get(level);
        sumCount[0] += root.val;
        sumCount[1]++;
        levelOrderBottom(root.left, level + 1, res);
        levelOrderBottom(root.right, level + 1, res);
    }
```



#[486. 预测赢家](https://leetcode-cn.com/problems/predict-the-winner/) 

>  给定一个表示分数的非负整数数组。 玩家 1 从数组任意一端拿取一个分数，随后玩家 2 继续从剩余数组任意一端拿取分数，然后玩家 1 拿，…… 。每次一个玩家只能拿取一个分数，分数被拿取之后不再可取。直到没有剩余分数可取时游戏结束。最终获得分数总和最多的玩家获胜。
>
>  给定一个表示分数的数组，预测玩家1是否会成为赢家。你可以假设每个玩家的玩法都会使他的分数最大化。
>
>   
>
>  示例 1：
>
>  输入：[1, 5, 2]
>  输出：False
>  解释：一开始，玩家1可以从1和2中进行选择。
>  如果他选择 2（或者 1 ），那么玩家 2 可以从 1（或者 2 ）和 5 中进行选择。如果玩家 2 选择了 5 ，那么玩家 1 则只剩下 1（或者 2 ）可选。
>  所以，玩家 1 的最终分数为 1 + 2 = 3，而玩家 2 为 5 。
>  因此，玩家 1 永远不会成为赢家，返回 False 。
>  示例 2：
>
>  输入：[1, 5, 233, 7]
>  输出：True
>  解释：玩家 1 一开始选择 1 。然后玩家 2 必须从 5 和 7 中进行选择。无论玩家 2 选择了哪个，玩家 1 都可以选择 233 。
>       最终，玩家 1（234 分）比玩家 2（12 分）获得更多的分数，所以返回 True，表示玩家 1 可以成为赢家。
>
>
>  提示：
>
>  1 <= 给定的数组长度 <= 20.
>  数组里所有分数都为非负数且不会大于 10000000 。
>  如果最终两个玩家的分数相等，那么玩家 1 仍为赢家。

```java
 public boolean PredictTheWinner(int[] nums) {
         int sum = 0;
        for(int n : nums)
            sum += n;
        int len = nums.length;
        int[][] dp = new int[len][len];
        for(int i = 0; i < len; i++)
            dp[i][i] = nums[i];
        for(int j = 1; j < len; j++)
            dp[j-1][j] = Math.max(dp[j-1][j-1], dp[j][j]);
        // 按照对角线来递推
        for(int i = 2; i < len; i++)
            for(int row = 0; i + row < len; row++)
                dp[row][row+i]  = Math.max(nums[row] + Math.min(dp[row+1][i+row-1], dp[row+2][i+row]),
                        nums[i+row] + Math.min(dp[row][i+row-2], dp[row+1][i+row-1]));
        return dp[0][len-1] >= (sum - dp[0][len-1]);
    }
```



#[202. 快乐数](https://leetcode-cn.com/problems/happy-number/) 

>  编写一个算法来判断一个数 n 是不是快乐数。
>
>  「快乐数」定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。如果 可以变为  1，那么这个数就是快乐数。
>
>  如果 n 是快乐数就返回 True ；不是，则返回 False 。
>
>   
>
>  示例：
>
>  输入：19
>  输出：true
>  解释：
>  12 + 92 = 82
>  82 + 22 = 68
>  62 + 82 = 100
>  12 + 02 + 02 = 1

```java
 public boolean isHappy(int n) {
        //这个是实际测试的，如果想要形成循环，其中循环结果有4.
        //换言之当n等于4以后则意味着进入了死循环
        if(n==4){
            return false;
        }
        int result = 0;
        while(n>0){
            result += Math.pow(n%10,2);
            n = n/10;
        }
        //如果结果不是1则递归。
        return result==1?true:isHappy(result);
    }
```



#[1025. 除数博弈](https://leetcode-cn.com/problems/divisor-game/) 

>  爱丽丝和鲍勃一起玩游戏，他们轮流行动。爱丽丝先手开局。
>
>  最初，黑板上有一个数字 N 。在每个玩家的回合，玩家需要执行以下操作：
>
>  选出任一 x，满足 0 < x < N 且 N % x == 0 。
>  用 N - x 替换黑板上的数字 N 。
>  如果玩家无法执行这些操作，就会输掉游戏。
>
>  只有在爱丽丝在游戏中取得胜利时才返回 True，否则返回 False。假设两个玩家都以最佳状态参与游戏。
>
>   
>
>  示例 1：
>
>  输入：2
>  输出：true
>  解释：爱丽丝选择 1，鲍勃无法进行操作。
>  示例 2：
>
>  输入：3
>  输出：false
>  解释：爱丽丝选择 1，鲍勃也选择 1，然后爱丽丝无法进行操作。
>
>
>  提示：
>
>  1 <= N <= 1000

```java
public boolean divisorGame(int N) {
        return N % 2 == 0;
    }
```



#[877. 石子游戏](https://leetcode-cn.com/problems/stone-game/) 

>  亚历克斯和李用几堆石子在做游戏。偶数堆石子排成一行，每堆都有正整数颗石子 piles[i] 。
>
>  游戏以谁手中的石子最多来决出胜负。石子的总数是奇数，所以没有平局。
>
>  亚历克斯和李轮流进行，亚历克斯先开始。 每回合，玩家从行的开始或结束处取走整堆石头。 这种情况一直持续到没有更多的石子堆为止，此时手中石子最多的玩家获胜。
>
>  假设亚历克斯和李都发挥出最佳水平，当亚历克斯赢得比赛时返回 true ，当李赢得比赛时返回 false 。
>
>   
>
>  示例：
>
>  输入：[5,3,4,5]
>  输出：true
>  解释：
>  亚历克斯先开始，只能拿前 5 颗或后 5 颗石子 。
>  假设他取了前 5 颗，这一行就变成了 [3,4,5] 。
>  如果李拿走前 3 颗，那么剩下的是 [4,5]，亚历克斯拿走后 5 颗赢得 10 分。
>  如果李拿走后 5 颗，那么剩下的是 [3,4]，亚历克斯拿走后 4 颗赢得 9 分。
>  这表明，取前 5 颗石子对亚历克斯来说是一个胜利的举动，所以我们返回 true 。
>
>
>  提示：
>
>  2 <= piles.length <= 500
>  piles.length 是偶数。
>  1 <= piles[i] <= 500
>  sum(piles) 是奇数。

not best还有优化的空间

```java
 public boolean stoneGame(int[] piles) {
        int length = piles.length;
        int[] dp = new int[length];
        for (int i = 0; i < length; i++) {
            dp[i] = piles[i];
        }
        for (int i = length - 2; i >= 0; i--) {
            for (int j = i + 1; j < length; j++) {
                dp[j] = Math.max(piles[i] - dp[j], piles[j] - dp[j - 1]);
            }
        }
        return dp[length - 1] > 0;
    }
```



#[438. 找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/) 

>  
>  给定一个字符串 **s** 和一个非空字符串 **p**，找到 **s** 中所有是 **p** 的字母异位词的子串，返回这些子串的起始索引。
>
>  字符串只包含小写英文字母，并且字符串 **s** 和 **p** 的长度都不超过 20100。
>
>  **说明：**
>
>  - 字母异位词指字母相同，但排列不同的字符串。
>  - 不考虑答案输出的顺序。
>
>  **示例 1:**
>
>  ```
>  输入:
>  s: "cbaebabacd" p: "abc"
>  
>  输出:
>  [0, 6]
>  
>  解释:
>  起始索引等于 0 的子串是 "cba", 它是 "abc" 的字母异位词。
>  起始索引等于 6 的子串是 "bac", 它是 "abc" 的字母异位词。
>  ```
>
>   **示例 2:**
>
>  ```java
>  输入:
>  s: "abab" p: "ab"
>  
>  输出:
>  [0, 1, 2]
>  
>  解释:
>  起始索引等于 0 的子串是 "ab", 它是 "ab" 的字母异位词。
>  起始索引等于 1 的子串是 "ba", 它是 "ab" 的字母异位词。
>  起始索引等于 2 的子串是 "ab", 它是 "ab" 的字母异位词。
>  ```
>
>  

```java
  public List<Integer> findAnagrams(String s, String p) {
 				List<Integer> ans = new ArrayList<>();
        if (s == null || s.length() == 0 || s.length() < p.length()) {
            return ans;
        }
        char[] strP = p.toCharArray();
        char[] strS = s.toCharArray();
        //为什么要123 z = 122,97之前的肯定是用不到的
        int[] chS = new int[123];
        int[] chP = new int[123];
        //将p的字符串放进索引表里,同时将相同长度的s的字符也放进索引表里
        for (int i = 0; i < strP.length; i++) {
            chP[strP[i]]++;
            chS[strS[i]]++;
        }
        //开始遍历处理,之前已经处理的p长度的字符
        for (int i = 0; i <= strS.length - strP.length; i++) {
            boolean flag = true;
            //什么意思?i=0的时候,说明已经添加了一轮,要先把之前的一轮处理完
            //所以不需要走这个逻辑,在上个for循环中处理过了,只需要判断
            //i>0的时候,s的长度大于p,所以还需要控制指针
            if (i > 0) {
                //cbaebabacd
                //i = 1 chS[strS[0]='c' = 99] --
                //相当于指针后移一位,相当关键的两步
                chS[strS[i - 1]]--;
                //继续往hash表里添加数据
                chS[strS[i - 1 + strP.length]]++;
            }
            //判断索引位是否相等
            for (char j = 'a'; j <= 'z'; j++) {
                if (chS[j] != chP[j]) {
                    flag = false;
                    break;
                }
            }
            //满足
            if (flag) {
                ans.add(i);
            }
        }
        return ans;
    }
```



#[820. 单词的压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/) 

>  
>  给定一个单词列表，我们将这个列表编码成一个索引字符串 `S` 与一个索引列表 `A`。
>
>  例如，如果这个列表是 `["time", "me", "bell"]`，我们就可以将其表示为 `S = "time#bell#"` 和 `indexes = [0, 2, 5]`。
>
>  对于每一个索引，我们可以通过从字符串 `S` 中索引的位置开始读取字符串，直到 "#" 结束，来恢复我们之前的单词列表。
>
>  那么成功对给定单词列表进行编码的最小字符串长度是多少呢？
>
>   
>
>  **示例：**
>
>  ```
>  输入: words = ["time", "me", "bell"]
>  输出: 10
>  说明: S = "time#bell#" ， indexes = [0, 2, 5] 。
>  ```
>
>   
>
>  **提示：**
>
>  1. `1 <= words.length <= 2000`
>  2. `1 <= words[i].length <= 7`
>  3. 每个单词都是小写字母 。

```java
{
          int len = 0;
        Trie trie = new Trie();
        // 先对单词列表根据单词长度由长到短排序
        Arrays.sort(words, (s1, s2) -> s2.length() - s1.length());
        // 单词插入trie，返回该单词增加的编码长度
        for (String word: words) {
            len += trie.insert(word);
        }
        return len;
    }
    // 定义tire
class Trie {
    
    TrieNode root;
    
    public Trie() {
        root = new TrieNode();
    }

    public int insert(String word) {
        TrieNode cur = root;
        boolean isNew = false;
        // 倒着插入单词
        for (int i = word.length() - 1; i >= 0; i--) {
            int c = word.charAt(i) - 'a';
            if (cur.children[c] == null) {
                isNew = true; // 是新单词
                cur.children[c] = new TrieNode();
            }
            cur = cur.children[c];
        }
        // 如果是新单词的话编码长度增加新单词的长度+1，否则不变。
        return isNew? word.length() + 1: 0;
    }
}

class TrieNode {
    char val;
    TrieNode[] children = new TrieNode[26];

    public TrieNode() {}
}
```



#[863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/) 

>  给定一个二叉树（具有根结点 root）， 一个目标结点 target ，和一个整数值 K 。
>
>  返回到目标结点 target 距离为 K 的所有结点的值的列表。 答案可以以任何顺序返回。
>
>   
>
>  示例 1：
>
>  输入：root = [3,5,1,6,2,0,8,null,null,7,4], target = 5, K = 2
>  输出：[7,4,1]
>  解释：
>  所求结点为与目标结点（值为 5）距离为 2 的结点，
>  值分别为 7，4，以及 1
>
>  
>
>  注意，输入的 "root" 和 "target" 实际上是树上的结点。
>  上面的输入仅仅是对这些对象进行了序列化描述。
>
>
>  提示：
>
>  给定的树是非空的。
>  树上的每个结点都具有唯一的值 0 <= node.val <= 500 。
>  目标结点 target 是树上的结点。
>  0 <= K <= 1000.

```java
 Map<TreeNode, TreeNode> parent;
    public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
        parent = new HashMap();
        dfs(root, null);

        Queue<TreeNode> queue = new LinkedList();
        queue.add(null);
        queue.add(target);

        Set<TreeNode> seen = new HashSet();
        seen.add(target);
        seen.add(null);

        int dist = 0;
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node == null) {
                if (dist == K) {
                    List<Integer> ans = new ArrayList();
                    for (TreeNode n: queue)
                        ans.add(n.val);
                    return ans;
                }
                queue.offer(null);
                dist++;
            } else {
                if (!seen.contains(node.left)) {
                    seen.add(node.left);
                    queue.offer(node.left);
                }
                if (!seen.contains(node.right)) {
                    seen.add(node.right);
                    queue.offer(node.right);
                }
                TreeNode par = parent.get(node);
                if (!seen.contains(par)) {
                    seen.add(par);
                    queue.offer(par);
                }
            }
        }

        return new ArrayList<Integer>();
    }

    public void dfs(TreeNode node, TreeNode par) {
        if (node != null) {
            parent.put(node, par);
            dfs(node.left, node);
            dfs(node.right, node);
        }
    }
```



#[242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/) 

>  给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。
>
>  示例 1:
>
>  输入: s = "anagram", t = "nagaram"
>  输出: true
>  示例 2:
>
>  输入: s = "rat", t = "car"
>  输出: false
>  说明:
>  你可以假设字符串只包含小写字母。
>
>  进阶:
>  如果输入字符串包含 unicode 字符怎么办？你能否调整你的解法来应对这种情况？

```java
public boolean isAnagram(String s, String t) {
          if (s.length() != t.length()) {
            return false;
        }
        int freq[] = new int[26];
        int fre[] = new int[26];
        for (int i = 0; i < s.length(); i++)
            freq[s.charAt(i) - 'a']++;
        for (int i = 0; i < t.length(); i++)
            fre[t.charAt(i) - 'a']++;
        //判断数组是否相等
        return Arrays.equals(fre, freq);
    }
```



#[338. 比特位计数](https://leetcode-cn.com/problems/counting-bits/) 

>  给定一个非负整数 num。对于 0 ≤ i ≤ num 范围中的每个数字 i ，计算其二进制数中的 1 的数目并将它们作为数组返回。
>
>  示例 1:
>
>  输入: 2
>  输出: [0,1,1]
>  示例 2:
>
>  输入: 5
>  输出: [0,1,1,2,1,2]
>  进阶:
>
>  给出时间复杂度为O(n*sizeof(integer))的解答非常容易。但你可以在线性时间O(n)内用一趟扫描做到吗？
>  要求算法的空间复杂度为O(n)。
>  你能进一步完善解法吗？要求在C++或任何其他语言中不使用任何内置函数（如 C++ 中的 __builtin_popcount）来执行此操作。

```java
 public int[] countBits(int num) {
    int dp[] = new int[num+1];
    for (int i = 0; i <= num/2; i++) {
        dp[i*2] = dp[i];
        if(i*2+1 <= num)
            dp[i*2+1] = dp[i] + 1;
    }
    return dp;
 }
```



#[845. 数组中的最长山脉](https://leetcode-cn.com/problems/longest-mountain-in-array/) 

>  
>  我们把数组 A 中符合下列属性的任意连续子数组 B 称为 “*山脉”*：
>
>  - `B.length >= 3`
>  - 存在 `0 < i < B.length - 1` 使得 `B[0] < B[1] < ... B[i-1] < B[i] > B[i+1] > ... > B[B.length - 1]`
>
>  （注意：B 可以是 A 的任意子数组，包括整个数组 A。）
>
>  给出一个整数数组 `A`，返回最长 *“山脉”* 的长度。
>
>  如果不含有 “*山脉”* 则返回 `0`。
>
>   
>
>  **示例 1：**
>
>  ```
>  输入：[2,1,4,7,3,2,5]
>  输出：5
>  解释：最长的 “山脉” 是 [1,4,7,3,2]，长度为 5。
>  ```
>
>  **示例 2：**
>
>  ```
>  输入：[2,2,2]
>  输出：0
>  解释：不含 “山脉”。
>  ```
>
>   
>
>  **提示：**
>
>  1. `0 <= A.length <= 10000`
>  2. `0 <= A[i] <= 10000`

```java
 public int longestMountain(int[] A) {
         if (A == null || A.length <= 2) {
            return 0;
        }
        int res = 0;
        for (int i = 1; i < A.length - 1; i++) {
            if (A[i - 1] < A[i] && A[i + 1] < A[i]) {
                int l = i - 1;
                int r = i + 1;
                while (l > 0 && A[l - 1] < A[l]) {
                    l--;
                }
                while (r < A.length - 1 && A[r + 1] < A[r]) {
                    r++;
                }
                res = Math.max(res, (r - l + 1));
            }
        }
        return res;
    }
```



#[474. 一和零](https://leetcode-cn.com/problems/ones-and-zeroes/) 

>  给你一个二进制字符串数组 strs 和两个整数 m 和 n 。
>
>  请你找出并返回 strs 的最大子集的大小，该子集中 最多 有 m 个 0 和 n 个 1 。
>
>  如果 x 的所有元素也是 y 的元素，集合 x 是集合 y 的 子集 。
>
>   
>
>  示例 1：
>
>  输入：strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3
>  输出：4
>  解释：最多有 5 个 0 和 3 个 1 的最大子集是 {"10","0001","1","0"} ，因此答案是 4 。
>  其他满足题意但较小的子集包括 {"0001","1"} 和 {"10","1","0"} 。{"111001"} 不满足题意，因为它含 4 个 1 ，大于 n 的值 3 。
>  示例 2：
>
>  输入：strs = ["10", "0", "1"], m = 1, n = 1
>  输出：2
>  解释：最大的子集是 {"0", "1"} ，所以答案是 2 。
>
>
>  提示：
>
>  1 <= strs.length <= 600
>  1 <= strs[i].length <= 100
>  strs[i] 仅由 '0' 和 '1' 组成
>  1 <= m, n <= 100

```java
public int findMaxForm(String[] strs, int m, int n) {
        int[][] dp = new int[m+1][n+1];
        for(int i = 0; i < strs.length; ++i) {
            int zero = 0, one = 0;
            for(int j = 0; j < strs[i].length(); ++j) {
                if(strs[i].charAt(j) == '0') ++zero;
                else ++one;
            }
            for(int j = m; j >= zero; --j) {
                for(int k = n; k >= one; --k) {
                    int tmp = dp[j-zero][k-one] + 1;
                    if(dp[j][k] < tmp)
                        dp[j][k] = tmp;
                }
            }
        }
        return dp[m][n];
    }
```

