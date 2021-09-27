[toc]

#[剑指 Offer 33 - 二叉搜索树的后序遍历](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/) 

>  输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。
>
>   
>
>  参考以下这颗二叉搜索树：
>
>       5
>      / \
>     2   6
>    / \
>   1   3
>  示例 1：
>
>  输入: [1,6,3,2,5]
>  输出: false
>  示例 2：
>
>  输入: [1,3,2,6,5]
>  输出: true

```java
  public boolean verifyPostorder(int[] postorder) {
        if (postorder.length == 0) {
            return true;
        }
        return verify(postorder, 0, postorder.length -1);
    }
    private boolean verify(int [] seq, int start, int end) {
         if (start >= end) {
             return true;
         }
         int i = start, j = end - 1;
         while (i < end && seq[i] < seq[end]) {
             i ++;
         }
         while (j >= start && seq[j] > seq[end]) {
             j --;
         }
         if (i < j) {
             return false;
         }
         return verify(seq, start, i - 1) && verify(seq, j + 1, end - 1);
    }
```



#[剑指 Offer 45 - 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

>  输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。
>
>   
>
>  示例 1:
>
>  输入: [10,2]
>  输出: "102"
>  示例 2:
>
>  输入: [3,30,34,5,9]
>  输出: "3033459"
>
>
>  提示:
>
>  0 < nums.length <= 100
>  说明:
>
>  输出结果可能非常大，所以你需要返回一个字符串而不是整数
>  拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0

```java
public String minNumber(int[] nums) {
         List<String> list = new ArrayList<>();
        for (int num : nums) {
            list.add(String.valueOf(num));
        }
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return (o1 + o2).compareTo(o2 + o1);
            }
        });
        return String.join("",list);
    }
```



# [剑指 Offer 14- I - 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

>  
>  给你一根长度为 `n` 的绳子，请把绳子剪成整数长度的 `m` 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 `k[0],k[1]...k[m-1]` 。请问 `k[0]*k[1]*...*k[m-1]` 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。
>
>  **示例 1：**
>
>  ```
>  输入: 2
>  输出: 1
>  解释: 2 = 1 + 1, 1 × 1 = 1
>  ```
>
>  **示例 2:**
>
>  ```
>  输入: 10
>  输出: 36
>  解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
>  ```
>
>  **提示：**
>
>  - `2 <= n <= 58`

```java
  public int cuttingRope(int n) {
        if(n == 2) {
            return 1;
        }
        if(n == 3){
            return 2;
        }
        int mod = (int)1e9 + 7;
        long res = 1;
        while(n > 4) {
            res *= 3;
            res %= mod;
            n -= 3;
        }
        return (int)(res * n % mod);
    }
```



# [90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

>  给定一个可能包含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。
>
>  说明：解集不能包含重复的子集。
>
>  示例:
>
>  输入: [1,2,2]
>  输出:
>  [
>    [2],
>    [1],
>    [1,2,2],
>    [2,2],
>    [1,2],
>    []
>  ]

```java
  public List<List<Integer>> subsetsWithDup(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        back(nums, res, 0, new ArrayList<>());
        return res;
    }

    private void back(int[] nums, List<List<Integer>> res, int i, ArrayList<Integer> tmp) {
        // TODO Auto-generated method stub
        res.add(new ArrayList<>(tmp));
        for (int j = i; j < nums.length; j++) {
            if (j > i && nums[j] == nums[j - 1]) {
                continue;
            }
            tmp.add(nums[j]);
            back(nums, res, j + 1, tmp);
            tmp.remove(tmp.size() - 1);
        }
    }
```



#[107. 二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

>  给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）
>
>  例如：
>  给定二叉树 [3,9,20,null,null,15,7],
>
>      3
>     / \
>    9  20
>      /  \
>     15   7
>  返回其自底向上的层次遍历为：
>
>  [
>    [15,7],
>    [9,20],
>    [3]
>  ]

```java
public List<List<Integer>> levelOrderBottom(TreeNode root) {
        if(root == null) return Collections.emptyList();
        List<List<Integer>> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
           
           //存放当前的一行
           List<Integer> level = new ArrayList<>();
         
           int size = queue.size();
           for(int i = 0;i < size;i++){
                TreeNode cur =  queue.poll();
                TreeNode left = cur.left,right = cur.right;
                level.add(cur.val);
                if(left != null) queue.offer(left);
                if(right != null) queue.offer(right);
           }
            res.add(0,level);
        }
        return res;
    }
```



#[673. 最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)

>  给定一个未排序的整数数组，找到最长递增子序列的个数。
>
>  示例 1:
>
>  输入: [1,3,5,4,7]
>  输出: 2
>  解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。
>  示例 2:
>
>  输入: [2,2,2,2,2]
>  输出: 5
>  解释: 最长递增子序列的长度是1，并且存在5个子序列的长度为1，因此输出5。
>  注意: 给定的数组长度不超过 2000 并且结果一定是32位有符号整数。

```java
public int findNumberOfLIS(int[] nums) {
        int N = nums.length;
        if (N <= 0) return N;
        int[] sort = nums.clone();
        Arrays.sort(sort);
        Map<Integer, Integer> map = new HashMap<>();
        // serialize
        for (int i = 0; i < N; i++) map.put(sort[i], i + 1);

        BITree tree = new BITree(N);
        for (int num: nums) {
            int id = map.get(num);
            int[] q = tree.query(id - 1);
            tree.update(id, q[0] + 1, q[1]);
        }
        return tree.query(N)[1];
    }
}

class BITree {
    int[] sums;
    int[] lens;

    public BITree(int n) {
        sums = new int[n + 1];
        lens = new int[n + 1];
    }
    
    // update counter of length with x
    public void update(int id, int x, int c) {
        for (int i = id; i < sums.length; i += lowbit(i)) {
            if (lens[i] == x) {
                sums[i] += c;
            } else if (lens[i] < x) {
                lens[i] = x;
                sums[i] = c;
            }
        }
    }

    // find the amount of longest subsequence exist before
    public int[] query(int id) {
        int x = 0;
        int c = 1;
        for (int i = id; i >= 1; i -= lowbit(i)) {
            if (lens[i] > x) {
                c = sums[i];
                x = lens[i];
            } else if (lens[i] == x) {
                c += sums[i];
            }
        }
        return new int[] {x, c};
    }

    public int lowbit(int x) {
        return x & (- x);
    }
```



#[149. 直线上最多的点数](https://leetcode-cn.com/problems/max-points-on-a-line/)

>  给定一个二维平面，平面上有 n 个点，求最多有多少个点在同一条直线上。
>
>  示例 1:
>
>  输入: [[1,1],[2,2],[3,3]]
>  输出: 3
>  解释:
>  ^
>  |
>  |        o
>  |     o
>  |  o  
>  +------------->
>  0  1  2  3  4
>  示例 2:
>
>  输入: [[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]
>  输出: 4
>  解释:
>  ^
>  |
>  |  o
>  |     o        o
>  |        o
>  |  o        o
>  +------------------->
>  0  1  2  3  4  5  6

```java
 public int maxPoints(int[][] points) {
        int n = points.length;
        if (n == 0) return 0;
        if (n == 1) return 1;
        int res = 0;
        for (int i = 0; i < n - 1; i++) {
            Map<String, Integer> slope = new HashMap<>();
            int repeat = 0;
            int tmp_max = 0;
            for (int j = i + 1; j < n; j++) {
                int dy = points[i][1] - points[j][1];
                int dx = points[i][0] - points[j][0];
                if (dy == 0 && dx == 0) {
                    repeat++;
                    continue;
                }
                int g = gcd(dy, dx);
                if (g != 0) {
                    dy /= g;
                    dx /= g;
                }
                String tmp = String.valueOf(dy) + "/" + String.valueOf(dx);
                slope.put(tmp, slope.getOrDefault(tmp, 0) + 1);
                tmp_max = Math.max(tmp_max, slope.get(tmp));
            }
            res = Math.max(res, repeat + tmp_max + 1);
        }
        return res;
    }

    private int gcd(int dy, int dx) {
        if (dx == 0) return dy;
        else return gcd(dx, dy % dx);
    }
```



#[337. 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/) 

>  在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。
>
>  计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。
>
>  示例 1:
>
>  输入: [3,2,3,null,3,null,1]
>
>       3
>      / \
>     2   3
>      \   \ 
>       3   1
>
>  输出: 7 
>  解释: 小偷一晚能够盗取的最高金额 = 3 + 3 + 1 = 7.
>  示例 2:
>
>  输入: [3,4,5,1,3,null,1]
>
>       3
>      / \
>     4   5
>    / \   \ 
>   1   3   1
>
>  输出: 9
>  解释: 小偷一晚能够盗取的最高金额 = 4 + 5 = 9.

```java
  	public int rob(TreeNode root) {
        int[] rootStatus = dfs(root);
        return Math.max(rootStatus[0], rootStatus[1]);
    }

    public int[] dfs(TreeNode node) {
        if (node == null) {
            return new int[]{0, 0};
        }
        int[] l = dfs(node.left);
        int[] r = dfs(node.right);
        int selected = node.val + l[1] + r[1];
        int notSelected = Math.max(l[0], l[1]) + Math.max(r[0], r[1]);
        return new int[]{selected, notSelected};
    }
```



#[182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/) 

>  编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。
>
>  示例：
>
>  +----+---------+
>  | Id | Email   |
>  +----+---------+
>  | 1  | a@b.com |
>  | 2  | c@d.com |
>  | 3  | a@b.com |
>  +----+---------+
>  根据以上输入，你的查询应返回以下结果：
>
>  +---------+
>  | Email   |
>  +---------+
>  | a@b.com |
>  +---------+

```java
# Write your MySQL query statement below
select Email from Person group by Email having count(1) > 1;
```



#[29. 两数相除](https://leetcode-cn.com/problems/divide-two-integers/) 

>  给定两个整数，被除数 dividend 和除数 divisor。将两数相除，要求不使用乘法、除法和 mod 运算符。
>
>  返回被除数 dividend 除以除数 divisor 得到的商。
>
>  整数除法的结果应当截去（truncate）其小数部分，例如：truncate(8.345) = 8 以及 truncate(-2.7335) = -2
>
>   
>
>  示例 1:
>
>  输入: dividend = 10, divisor = 3
>  输出: 3
>  解释: 10/3 = truncate(3.33333..) = truncate(3) = 3
>  示例 2:
>
>  输入: dividend = 7, divisor = -3
>  输出: -2
>  解释: 7/-3 = truncate(-2.33333..) = -2
>
>
>  提示：
>
>  被除数和除数均为 32 位有符号整数。
>  除数不为 0。
>  假设我们的环境只能存储 32 位有符号整数，其数值范围是 [−231,  231 − 1]。本题中，如果除法结果溢出，则返回 231 − 1。

```java
    public int divide(int dividend, int divisor) {
        if (dividend == 0) {
            return 0;
        }
        if (dividend == Integer.MIN_VALUE && divisor == -1) {
            return Integer.MAX_VALUE;
        }
        boolean negative;
        negative = (dividend ^ divisor) <0;//用异或来计算是否符号相异
        long t = Math.abs((long) dividend);
        long d= Math.abs((long) divisor);
        int result = 0;
        for (int i=31; i>=0;i--) {
            if ((t>>i)>=d) {//找出足够大的数2^n*divisor
                result+=1<<i;//将结果加上2^n
                t-=d<<i;//将被除数减去2^n*divisor
            }
        }
        return negative ? -result : result;//符号相异取反
    }
```



#[130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/) 

>  给定一个二维的矩阵，包含 'X' 和 'O'（字母 O）。
>
>  找到所有被 'X' 围绕的区域，并将这些区域里所有的 'O' 用 'X' 填充。
>
>  示例:
>
>  X X X X
>  X O O X
>  X X O X
>  X O X X
>  运行你的函数后，矩阵变为：
>
>  X X X X
>  X X X X
>  X X X X
>  X O X X
>  解释:
>
>  被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。
>

```java
   int[][] map = new int[][]{{-1,0},{0,1},{1,0},{0,-1}};
    boolean[][] visited;

    public void solve(char[][] board) {

        if(board.length == 0) return;

        visited = new boolean[board.length][board[0].length];
        for (int i = 0; i < board.length; i++) {
            for(int j=0; j<board[i].length; j++){
                visited[i][j] = false;
            }
        }

        //找出与边界相连的所有O,以及所有X，标记为true
        for (int i = 0; i < board.length; i++) {
            for(int j=0; j<board[i].length; j++){
                if(board[i][j]=='X') visited[i][j] = true;
                //从边界开始进行dfs
                if(board[i][j]=='O' && (i==0 || j==0 || i==board.length-1 || j==board[i].length-1)){
                    dfs(board, i, j);
                }
            }
        }
        //没有被标记的就是没有与边界相邻的O了
        for (int i = 0; i < visited.length; i++) {
            for(int j=0; j<visited[i].length; j++){
                if(!visited[i][j]){
                    board[i][j] = 'X';
                }
            }
        }

    }

    public void dfs(char[][] board, int startX, int startY){
    
        for(int i=0; i<4; i++){
            visited[startX][startY] = true;
            int newX = startX+map[i][0];
            int newY = startY+map[i][1];
            
            if(newX>0 && newY>0 && newX<board.length && newY<board[0].length && !visited[newX][newY]){
                if(board[newX][newY] == 'O'){
                    dfs(board, newX, newY);
                }
            }
        }
    }
```



#[166. 分数到小数](https://leetcode-cn.com/problems/fraction-to-recurring-decimal/) 

>  给定两个整数，分别表示分数的分子 numerator 和分母 denominator，以 字符串形式返回小数 。
>
>  如果小数部分为循环小数，则将循环的部分括在括号内。
>
>  如果存在多个答案，只需返回 任意一个 。
>
>  对于所有给定的输入，保证 答案字符串的长度小于 104 。
>
>   
>
>  示例 1：
>
>  输入：numerator = 1, denominator = 2
>  输出："0.5"
>  示例 2：
>
>  输入：numerator = 2, denominator = 1
>  输出："2"
>  示例 3：
>
>  输入：numerator = 2, denominator = 3
>  输出："0.(6)"
>  示例 4：
>
>  输入：numerator = 4, denominator = 333
>  输出："0.(012)"
>  示例 5：
>
>  输入：numerator = 1, denominator = 5
>  输出："0.2"
>
>
>  提示：
>
>  -231 <= numerator, denominator <= 231 - 1
>  denominator != 0

```java
 public String fractionToDecimal(int numerator, int denominator) {
                if (numerator == 0) {
            return "0";
        }
        StringBuilder fraction = new StringBuilder();
        // If either one is negative (not both)
        if (numerator < 0 ^ denominator < 0) {
            fraction.append("-");
        }
        // Convert to Long or else abs(-2147483648) overflows
        long dividend = Math.abs(Long.valueOf(numerator));
        long divisor = Math.abs(Long.valueOf(denominator));
        fraction.append(String.valueOf(dividend / divisor));
        long remainder = dividend % divisor;
        if (remainder == 0) {
            return fraction.toString();
        }
        fraction.append(".");
        Map<Long, Integer> map = new HashMap<>();
        while (remainder != 0) {
            if (map.containsKey(remainder)) {
                fraction.insert(map.get(remainder), "(");
                fraction.append(")");
                break;
            }
            map.put(remainder, fraction.length());
            remainder *= 10;
            fraction.append(remainder / divisor);
            remainder %= divisor;
        }
        return fraction.toString();
    }
```



#[216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/) 

>  找出所有相加之和为 n 的 k 个数的组合。组合中只允许含有 1 - 9 的正整数，并且每种组合中不存在重复的数字。
>
>  说明：
>
>  所有数字都是正整数。
>  解集不能包含重复的组合。 
>  示例 1:
>
>  输入: k = 3, n = 7
>  输出: [[1,2,4]]
>  示例 2:
>
>  输入: k = 3, n = 9
>  输出: [[1,2,6], [1,3,5], [2,3,4]]

```java
 public List<List<Integer>> combinationSum3(int k, int n) {
		List<List<Integer>> res = new ArrayList<>();
		List<Integer> cur = new ArrayList<>();
		dfs(1,0,k,n,res,cur);
		return res;
    }
	private void dfs(int i,int sum,int k,int n,List<List<Integer>> res,List<Integer> cur){
		//base case
		if(k == 0 && sum == n){
			res.add(new ArrayList<>(cur));
		}
		if(sum > n) return;
		for(int j = i;j <= (Math.min(9,n));j++){
			cur.add(j);
			dfs(j+1,sum+j,k-1,n,res,cur);
			cur.remove(cur.size()-1);
		}
	}
```



#[49. 字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/) 

>  给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。
>
>  示例:
>
>  输入: ["eat", "tea", "tan", "ate", "nat", "bat"]
>  输出:
>  [
>    ["ate","eat","tea"],
>    ["nat","tan"],
>    ["bat"]
>  ]
>  说明：
>
>  所有输入均为小写字母。
>  不考虑答案输出的顺序。

```java
 public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String,ArrayList<String>> map=new HashMap<>();
        for(String s:strs){
            char[] ch=s.toCharArray();
            Arrays.sort(ch);
            String key=String.valueOf(ch);
            if(!map.containsKey(key))    map.put(key,new ArrayList<>());
            map.get(key).add(s);
        }
        return new ArrayList(map.values());
    }
```



#[238. 除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/) 

>  给你一个长度为 n 的整数数组 nums，其中 n > 1，返回输出数组 output ，其中 output[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积。
>
>   
>
>  示例:
>
>  输入: [1,2,3,4]
>  输出: [24,12,8,6]
>
>
>  提示：题目数据保证数组之中任意元素的全部前缀元素和后缀（甚至是整个数组）的乘积都在 32 位整数范围内。
>
>  说明: 请不要使用除法，且在 O(n) 时间复杂度内完成此题。
>
>  进阶：
>  你可以在常数空间复杂度内完成这个题目吗？（ 出于对空间复杂度分析的目的，输出数组不被视为额外空间。）

```java
 public int[] productExceptSelf(int[] nums) {
        int left = 1;
        int right = 1;
        int len = nums.length;
        int[] dp = new int[len];
        for(int i = 0;i<len;i++){
            dp[i] = left;
            left *= nums[i];
        }
        for(int i = len -1;i>=0;i--){
            dp[i] *= right;
            right *= nums[i];
        }
        return dp;
    }
```



#[140. 单词拆分 II](https://leetcode-cn.com/problems/word-break-ii/) 

>  给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，在字符串中增加空格来构建一个句子，使得句子中所有的单词都在词典中。返回所有这些可能的句子。
>
>  说明：
>
>  分隔时可以重复使用字典中的单词。
>  你可以假设字典中没有重复的单词。
>  示例 1：
>
>  输入:
>  s = "catsanddog"
>  wordDict = ["cat", "cats", "and", "sand", "dog"]
>  输出:
>  [
>    "cats and dog",
>    "cat sand dog"
>  ]
>  示例 2：
>
>  输入:
>  s = "pineapplepenapple"
>  wordDict = ["apple", "pen", "applepen", "pine", "pineapple"]
>  输出:
>  [
>    "pine apple pen apple",
>    "pineapple pen apple",
>    "pine applepen apple"
>  ]
>  解释: 注意你可以重复使用字典中的单词。
>  示例 3：
>
>  输入:
>  s = "catsandog"
>  wordDict = ["cats", "dog", "sand", "and", "cat"]
>  输出:
>  []

```java
private int maxLength = -1;
    private Set<String> wordSet = new HashSet<>();
    private Map<Integer, List <String>> mapPosToStrings = new HashMap<>();

    List<String> calculateResult(String s, int currentPos) {
        if (currentPos == s.length()) {
            List<String> result = new ArrayList<>();
            result.add("");
            return result;
        }
        if (mapPosToStrings.containsKey(currentPos)) {
            return mapPosToStrings.get(currentPos);
        }
        List<String> result = new ArrayList<>();
        mapPosToStrings.put(currentPos, result);

        for (int i = 1; i <= maxLength && currentPos + i <= s.length(); i++) {
            String subString = s.substring(currentPos, currentPos + i);
            if (wordSet.contains(subString)) {
                List<String> returnStrings = calculateResult(s, currentPos + i);
                for (String returnString: returnStrings) {
                    if (returnString.equals("")) {
                        result.add(subString);
                    } else {
                        result.add(subString + " " + returnString);
                    }
                }
            }
        }
        return result;
    }

    public List<String> wordBreak(String s, List<String> wordDict) {
        for (String word: wordDict) {
            wordSet.add(word);
            maxLength = Math.max(maxLength, word.length());
        }

        return calculateResult(s, 0);
    }
```





#[678. 有效的括号字符串](https://leetcode-cn.com/problems/valid-parenthesis-string/) 

>  给定一个只包含三种字符的字符串：（ ，） 和 *，写一个函数来检验这个字符串是否为有效字符串。有效字符串具有如下规则：
>
>  任何左括号 ( 必须有相应的右括号 )。
>  任何右括号 ) 必须有相应的左括号 ( 。
>  左括号 ( 必须在对应的右括号之前 )。
>  * 可以被视为单个右括号 ) ，或单个左括号 ( ，或一个空字符串。
>  一个空字符串也被视为有效字符串。
>  示例 1:
>
>  输入: "()"
>  输出: True
>  示例 2:
>
>  输入: "(*)"
>  输出: True
>  示例 3:
>
>  输入: "(*))"
>  输出: True
>  注意:
>
>  字符串大小将在 [1，100] 范围内。
>

```java
 public boolean checkValidString(String s) {
        Deque<Integer> leftStack = new ArrayDeque<>();
        Deque<Integer> starStack = new ArrayDeque<>();
        int len = s.length();
        for(int i = 0; i < len; i++){
            char ch = s.charAt(i);
            if(ch == ')'){
                //左括号如果为空，那么利用 * 来替代
                if(leftStack.isEmpty()){
                    //如果 * 不为空
                    if(!starStack.isEmpty()){
                        starStack.pop();
                    }else{
                        //如果 * 为空，表示莫得匹配
                        return false;
                    }
                }else{
                    //左括号不为空
                    leftStack.pop();
                }
            }else if(ch == '('){
                leftStack.push(i);
            }else{
                starStack.push(i);
            }
        }
        //
        while(!leftStack.isEmpty() && !starStack.isEmpty()){
            //当左括号的索引小于 * 号索引，即 * 号可以用来替代右括号
            if(leftStack.peek() < starStack.peek()){
                leftStack.pop();
                starStack.pop();
            }else{
                break;
            }
        }
        return leftStack.isEmpty();
    }
```



#[680. 验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/) 

>  给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。
>
>  示例 1:
>
>  输入: "aba"
>  输出: True
>  示例 2:
>
>  输入: "abca"
>  输出: True
>  解释: 你可以删除c字符。
>  注意:
>
>  字符串只包含从 a-z 的小写字母。字符串的最大长度是50000。
>

```java
 public boolean validPalindrome(String s) {
        int i = 0, j = s.length() - 1;
        while(i < j){
            if(s.charAt(i) != s.charAt(j)){
                return isValid(s, i + 1, j) || isValid(s, i, j - 1);
            }
            i++;
            j--;
        }
        return true;
    }
    
    public boolean isValid(String s, int i, int j){
        while(i < j){
            if(s.charAt(i) != s.charAt(j)){
                return false;
            }
            i++;
            j--;
        }
        return true;
    }
```



#[剑指 Offer 49 - 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/) 

>  我们把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。
>
>   
>
>  示例:
>
>  输入: n = 10
>  输出: 12
>  解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
>  说明:  
>
>  1 是丑数。
>  n 不超过1690。
>  注意：本题与主站 264 题相同：https://leetcode-cn.com/problems/ugly-number-ii/

```java
public int nthUglyNumber(int n) {
         if (n <= 0)
            return -1;
        int[] dp = new int[n];
        dp[0] = 1;
        int id2 = 0, id3 = 0, id5 = 0;
        for (int i = 1; i < n; i++) {
            dp[i] = Math.min(dp[id2] * 2, Math.min(dp[id3] *3, dp[id5] * 5));
            // 这里不用else if的原因是有可能id2(3) * 2 == id3(2) * 3
            // 这种情况两个指针都要后移
            if (dp[id2] * 2 == dp[i])
                id2 += 1;
            if (dp[id3] * 3 == dp[i])
                id3 += 1;
            if (dp[id5] * 5 == dp[i])
                id5 += 1; 
        }
        return dp[n - 1];
    }
```

# [977. 有序数组的平方](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)

>  给定一个按非递减顺序排序的整数数组 A，返回每个数字的平方组成的新数组，要求也按非递减顺序排序。
>
>   
>
>  示例 1：
>
>  输入：[-4,-1,0,3,10]
>  输出：[0,1,9,16,100]
>  示例 2：
>
>  输入：[-7,-3,2,3,11]
>  输出：[4,9,9,49,121]
>
>
>  提示：
>
>  1 <= A.length <= 10000
>  -10000 <= A[i] <= 10000
>  A 已按非递减顺序排序。

```java
public int[] sortedSquares(int[] A) {
       int N = A.length;
        int j = 0;
        while (j < N && A[j] < 0)
            j++;
        int i = j - 1;

        int[] ans = new int[N];
        int t = 0;

        while (i >= 0 && j < N) {
            if (A[i] * A[i] < A[j] * A[j]) {
                ans[t++] = A[i] * A[i];
                i--;
            } else {
                ans[t++] = A[j] * A[j];
                j++;
            }
        }

        while (i >= 0) {
            ans[t++] = A[i] * A[i];
            i--;
        }
        while (j < N) {
            ans[t++] = A[j] * A[j];
            j++;
        }

        return ans;
    }
```

