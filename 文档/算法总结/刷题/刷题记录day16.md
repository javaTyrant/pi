[toc]

# [面试题 01.06 - 字符串压缩](https://leetcode-cn.com/problems/compress-string-lcci/)

>  字符串压缩。利用字符重复出现的次数，编写一种方法，实现基本的字符串压缩功能。比如，字符串aabcccccaaa会变为a2b1c5a3。若“压缩”后的字符串没有变短，则返回原先的字符串。你可以假设字符串中只包含大小写英文字母（a至z）。
>
>  示例1:
>
>   输入："aabcccccaaa"
>   输出："a2b1c5a3"
>  示例2:
>
>   输入："abbccd"
>   输出："abbccd"
>   解释："abbccd"压缩后为"a1b2c2d1"，比原字符串长度更长。
>  提示：
>
>  字符串长度在[0, 50000]范围内。
>

```java
   public String compressString(String S) {
         if (S == null || S.length() <= 2)
            return S;
        int len = S.length();
        int cnt = 1;
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i < len; i++)
            if (S.charAt(i) == S.charAt(i - 1))
                cnt++;
            else {
                sb.append(S.charAt(i - 1)).append(cnt);
                cnt = 1;
            }
        sb.append(S.charAt(len - 1)).append(cnt);
        String ans = sb.toString();
        return ans.length() < len ? ans : S;
    }
```



#[剑指 Offer 47 - 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/) 

>  在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？
>
>   
>
>  示例 1:
>
>  输入: 
>  [
>    [1,3,1],
>    [1,5,1],
>    [4,2,1]
>  ]
>  输出: 12
>  解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
>
>
>  提示：
>
>  0 < grid.length <= 200
>  0 < grid[0].length <= 200

```java
public int maxValue(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int []dp = new int[n];
        for(int i = 0; i < m; i++){
            dp[0] = dp[0] + grid[i][0];
            for(int j = 1; j < n; j++){
                dp[j] = Math.max(dp[j], dp[j-1]) + grid[i][j];
            }
        }
        return dp[n-1];
    }
```



#[剑指 Offer 37 - 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/) 

>  请实现两个函数，分别用来序列化和反序列化二叉树。
>
>  示例: 
>
>  你可以将以下二叉树：
>
>      1
>     / \
>    2   3
>       / \
>      4   5
>
>  序列化为 "[1,2,3,null,null,4,5]"
>
>  **说明:** 不要使用类的成员 / 全局 / 静态变量来存储状态，你的序列化和反序列化算法应该是无状态的。

```java
	//序列化dfs
	public List<Integer> serialize(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        dfs(res, root);
        return res;
    }

    private void dfs(List<Integer> res, TreeNode root) {
        if (root == null) {
            res.add(null);
        } else {
            res.add(root.val);
            dfs(res, root.left);
            dfs(res, root.right);
        }
    }
		//反序列化
    public TreeNode deserialize(List<Integer> data) {
        int index[] = {0};
        TreeNode root = build(index, data);
        return root;
    }

    private TreeNode build(int[] index, List<Integer> data) {
        Integer val = data.get(index[0]);
        index[0] = index[0] + 1;
        if (val == null) return null;
        TreeNode node = new TreeNode(val);
        node.left = build(index, data);
        node.right = build(index, data);
        return node;
    }
```



#[459. 重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/) 

>  给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。
>
>  示例 1:
>
>  输入: "abab"
>
>  输出: True
>
>  解释: 可由子字符串 "ab" 重复两次构成。
>  示例 2:
>
>  输入: "aba"
>
>  输出: False
>  示例 3:
>
>  输入: "abcabcabcabc"
>
>  输出: True
>
>  解释: 可由子字符串 "abc" 重复四次构成。 (或者子字符串 "abcabc" 重复两次构成。)
>

```java
 public boolean repeatedSubstringPattern(String s) {
        if (s == null || s.length() <= 1) {
            return false;
        }
        int n = s.length();
        int[] prims = new int[]{
                2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97
        };

        for (int p : prims) {
            if (n % p != 0) {
                continue;
            }

            boolean flag = true;
            String sub = s.substring(0, n / p);
            for (int j = 1; j < p; j++) {
                if (!sub.equals(s.substring(n / p * j, n / p * (j + 1)))) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                return true;
            }
        }
        return false;
    }
```

# [1191. K 次串联后最大子数组之和](https://leetcode-cn.com/problems/k-concatenation-maximum-sum/)

>  给你一个整数数组 arr 和一个整数 k。
>
>  首先，我们要对该数组进行修改，即把原数组 arr 重复 k 次。
>
>  举个例子，如果 arr = [1, 2] 且 k = 3，那么修改后的数组就是 [1, 2, 1, 2, 1, 2]。
>
>  然后，请你返回修改后的数组中的最大的子数组之和。
>
>  注意，子数组长度可以是 0，在这种情况下它的总和也是 0。
>
>  由于 结果可能会很大，所以需要 模（mod） 10^9 + 7 后再返回。 
>
>   
>
>  示例 1：
>
>  输入：arr = [1,2], k = 3
>  输出：9
>  示例 2：
>
>  输入：arr = [1,-2,1], k = 5
>  输出：2
>  示例 3：
>
>  输入：arr = [-1,-2], k = 7
>  输出：0
>
>
>  提示：
>
>  1 <= arr.length <= 10^5
>  1 <= k <= 10^5
>  -10^4 <= arr[i] <= 10^4

```java
 public int kConcatenationMaxSum(int[] arr, int k) {
         int dp = 0, max = 0, sum = 0;
        for(int i = 0; i < 2 * arr.length; i++) {
            if(dp < 0)
                dp = arr[i % arr.length];
            else
                dp += arr[i % arr.length];
            max = Math.max(max, dp);
            if(i == arr.length - 1 && k == 1)
                return max;
            if(i < arr.length)
                sum += arr[i];
        }
        return (int)((Math.max(0, sum) * (k-2) + max) % 1000000007);
    }
```



#[958. 二叉树的完全性检验](https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/) 

>  给定一个二叉树，确定它是否是一个完全二叉树。
>
>  百度百科中对完全二叉树的定义如下：
>
>  若设二叉树的深度为 h，除第 h 层外，其它各层 (1～h-1) 的结点数都达到最大个数，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。（注：第 h 层可能包含 1~ 2h 个节点。）
>
>   
>
>  示例 1：
>
>  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/complete-binary-tree-1.png)
>
>  输入：[1,2,3,4,5,6]
>  输出：true
>  解释：最后一层前的每一层都是满的（即，结点值为 {1} 和 {2,3} 的两层），且最后一层中的所有结点（{4,5,6}）都尽可能地向左。
>  示例 2：
>
>  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/complete-binary-tree-2.png)
>
>  输入：[1,2,3,4,5,null,7]
>  输出：false
>  解释：值为 7 的结点没有尽可能靠向左侧。
>
>
>  提示：
>
>  树中将会有 1 到 100 个结点。
>

```java
public boolean isCompleteTree(TreeNode root) {
           if (root == null) return true;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        TreeNode temp;
        boolean flag = false;
        while (!queue.isEmpty()) {
            temp = queue.remove();
            if (temp == null){
                flag = true;
                continue;
            }
            if (flag) return false;
            queue.add(temp.left);
            queue.add(temp.right);
        }
        return true;
    }
```



#[393. UTF-8 编码验证](https://leetcode-cn.com/problems/utf-8-validation/) 

>  UTF-8 中的一个字符可能的长度为 1 到 4 字节，遵循以下的规则：
>
>  对于 1 字节的字符，字节的第一位设为0，后面7位为这个符号的unicode码。
>  对于 n 字节的字符 (n > 1)，第一个字节的前 n 位都设为1，第 n+1 位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的unicode码。
>  这是 UTF-8 编码的工作方式：
>
>     Char. number range  |        UTF-8 octet sequence
>        (hexadecimal)    |              (binary)
>     --------------------+---------------------------------------------
>     0000 0000-0000 007F | 0xxxxxxx
>     0000 0080-0000 07FF | 110xxxxx 10xxxxxx
>     0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
>     0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
>  给定一个表示数据的整数数组，返回它是否为有效的 utf-8 编码。
>
>  注意:
>  输入是整数数组。只有每个整数的最低 8 个有效位用来存储数据。这意味着每个整数只表示 1 字节的数据。
>
>  示例 1:
>
>  data = [197, 130, 1], 表示 8 位的序列: 11000101 10000010 00000001.
>
>  返回 true 。
>  这是有效的 utf-8 编码，为一个2字节字符，跟着一个1字节字符。
>  示例 2:
>
>  data = [235, 140, 4], 表示 8 位的序列: 11101011 10001100 00000100.
>
>  返回 false 。
>  前 3 位都是 1 ，第 4 位为 0 表示它是一个3字节字符。
>  下一个字节是开头为 10 的延续字节，这是正确的。
>  但第二个延续字节不以 10 开头，所以是不符合规则的。

```java
public boolean validUtf8(int[] data) {
      int cnt = 0;
        for(int d:data){
            if(cnt == 0){

                if((d >> 5) == 0b110) cnt = 1;
                else if((d >> 4) == 0b1110) cnt = 2;
                else if((d >> 3) == 0b11110) cnt = 3;
                else if((d >> 7) == 0b1) return false;
            }else {
                if((d >> 6) != 0b10) return false;
                cnt--;
            }
        }
        return cnt == 0;
    }
```



#[662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/) 

>  给定一个二叉树，编写一个函数来获取这个树的最大宽度。树的宽度是所有层中的最大宽度。这个二叉树与满二叉树（full binary tree）结构相同，但一些节点为空。
>
>  每一层的宽度被定义为两个端点（该层最左和最右的非空节点，两端点间的null节点也计入长度）之间的长度。
>
>  示例 1:
>
>  输入: 
>
>             1
>           /   \
>          3     2
>         / \     \  
>        5   3     9 
>
>  输出: 4
>  解释: 最大值出现在树的第 3 层，宽度为 4 (5,3,null,9)。
>  示例 2:
>
>  输入: 
>
>            1
>           /  
>          3    
>         / \       
>        5   3     
>
>  输出: 2
>  解释: 最大值出现在树的第 3 层，宽度为 2 (5,3)。
>  示例 3:
>
>  输入: 
>
>            1
>           / \
>          3   2 
>         /        
>        5      
>
>  输出: 2
>  解释: 最大值出现在树的第 2 层，宽度为 2 (3,2)。
>  示例 4:
>
>  输入: 
>
>            1
>           / \
>          3   2
>         /     \  
>        5       9 
>       /         \
>      6           7
>  输出: 8
>  解释: 最大值出现在树的第 4 层，宽度为 8 (6,null,null,null,null,null,null,7)。
>  注意: 答案在32位有符号整数的表示范围内。

```java
  int ans;
    Map<Integer, Integer> left;
    public int widthOfBinaryTree(TreeNode root) {
        ans = 0;
        left = new HashMap();
        dfs(root, 0, 0);
        return ans;
    }
    public void dfs(TreeNode root, int depth, int pos) {
        if (root == null) return;
        left.computeIfAbsent(depth, x-> pos);
        ans = Math.max(ans, pos - left.get(depth) + 1);
        dfs(root.left, depth + 1, 2 * pos);
        dfs(root.right, depth + 1, 2 * pos + 1);
    }
```



#[714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) 

>  给定一个整数数组 prices，其中第 i 个元素代表了第 i 天的股票价格 ；非负整数 fee 代表了交易股票的手续费用。
>
>  你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。
>
>  返回获得利润的最大值。
>
>  注意：这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。
>
>  示例 1:
>
>  输入: prices = [1, 3, 2, 8, 4, 9], fee = 2
>  输出: 8
>  解释: 能够达到的最大利润:  
>  在此处买入 prices[0] = 1
>  在此处卖出 prices[3] = 8
>  在此处买入 prices[4] = 4
>  在此处卖出 prices[5] = 9
>  总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
>  注意:
>
>  0 < prices.length <= 50000.
>  0 < prices[i] < 50000.
>  0 <= fee < 50000.

```java
public int maxProfit(int[] prices, int fee) {
        int cash = 0, hold = -prices[0];
        for (int i = 1; i < prices.length; i++) {
            cash = Math.max(cash, hold + prices[i] - fee);
            hold = Math.max(hold, cash - prices[i]);
        }
        return cash;
    }
```



#[876. 链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/) 

>  给定一个头结点为 head 的非空单链表，返回链表的中间结点。
>
>  如果有两个中间结点，则返回第二个中间结点。
>
>   
>
>  示例 1：
>
>  输入：[1,2,3,4,5]
>  输出：此列表中的结点 3 (序列化形式：[3,4,5])
>  返回的结点值为 3 。 (测评系统对该结点序列化表述是 [3,4,5])。
>  注意，我们返回了一个 ListNode 类型的对象 ans，这样：
>  ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, 以及 ans.next.next.next = NULL.
>  示例 2：
>
>  输入：[1,2,3,4,5,6]
>  输出：此列表中的结点 4 (序列化形式：[4,5,6])
>  由于该列表有两个中间结点，值分别为 3 和 4，我们返回第二个结点。
>
>
>  提示：
>
>  给定链表的结点数介于 1 和 100 之间。
>

```java
 public ListNode middleNode(ListNode head) {
        if(head == null){
            return null;
        }
        if(head.next == null){
            return head;
        }
        ListNode fast = head;
        ListNode slow = head;
        while(fast != null && fast.next != null){
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
```



#[292. Nim 游戏](https://leetcode-cn.com/problems/nim-game/) 哈哈哈

>  你和你的朋友，两个人一起玩 Nim 游戏：
>
>  桌子上有一堆石头。
>  你们轮流进行自己的回合，你作为先手。
>  每一回合，轮到的人拿掉 1 - 3 块石头。
>  拿掉最后一块石头的人就是获胜者。
>  假设你们每一步都是最优解。请编写一个函数，来判断你是否可以在给定石头数量为 n 的情况下赢得游戏。如果可以赢，返回 true；否则，返回 false 。
>
>   
>
>  示例 1：
>
>  输入：n = 4
>  输出：false 
>  解释：如果堆中有 4 块石头，那么你永远不会赢得比赛；
>       因为无论你拿走 1 块、2 块 还是 3 块石头，最后一块石头总是会被你的朋友拿走。
>  示例 2：
>
>  输入：n = 1
>  输出：true
>  示例 3：
>
>  输入：n = 2
>  输出：true
>
>
>  提示：
>
>  1 <= n <= 231 - 1

```java
 public boolean canWinNim(int n) {
        return n % 4 != 0;
    }
```



#[542. 01 矩阵](https://leetcode-cn.com/problems/01-matrix/) 

>  给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。
>
>  两个相邻元素间的距离为 1 。
>
>   
>
>  示例 1：
>
>  输入：
>  [[0,0,0],
>   [0,1,0],
>   [0,0,0]]
>
>  输出：
>  [[0,0,0],
>   [0,1,0],
>   [0,0,0]]
>  示例 2：
>
>  输入：
>  [[0,0,0],
>   [0,1,0],
>   [1,1,1]]
>
>  输出：
>  [[0,0,0],
>   [0,1,0],
>   [1,2,1]]
>
>
>  提示：
>
>  给定矩阵的元素个数不超过 10000。
>  给定矩阵中至少有一个元素是 0。
>  矩阵中的元素只在四个方向上相邻: 上、下、左、右。

```java
private int row;
    private int col;
    private int[][] vector = new int[][]{{0, 1}, {0, -1}, {1, 0}, {-1, 0}};

    /**
     * DP（两次遍历，可 AC）
     */
    public int[][] updateMatrix(int[][] matrix) {
        row = matrix.length;
        col = matrix[0].length;
        // 第一次遍历，正向遍历，根据相邻左元素和上元素得出当前元素的对应结果
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (matrix[i][j] == 1) {
                    matrix[i][j] = row + col;
                }
                if (i > 0) {
                    matrix[i][j] = Math.min(matrix[i][j], matrix[i - 1][j] + 1);
                }
                if (j > 0) {
                    matrix[i][j] = Math.min(matrix[i][j], matrix[i][j - 1] + 1);
                }
            }
        }
        // 第二次遍历，反向遍历，根据相邻右元素和下元素及当前元素的结果得出最终结果
        for (int i = row - 1; i >= 0; i--) {
            for (int j = col - 1; j >= 0; j--) {
                if (i < row - 1) {
                    matrix[i][j] = Math.min(matrix[i][j], matrix[i + 1][j] + 1);
                }
                if (j < col - 1) {
                    matrix[i][j] = Math.min(matrix[i][j], matrix[i][j + 1] + 1);
                }
            }
        }
        return matrix;
    }
```



#[452. 用最少数量的箭引爆气球](https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/) 

>  在二维空间中有许多球形的气球。对于每个气球，提供的输入是水平方向上，气球直径的开始和结束坐标。由于它是水平的，所以纵坐标并不重要，因此只要知道开始和结束的横坐标就足够了。开始坐标总是小于结束坐标。
>
>  一支弓箭可以沿着 x 轴从不同点完全垂直地射出。在坐标 x 处射出一支箭，若有一个气球的直径的开始和结束坐标为 xstart，xend， 且满足  xstart ≤ x ≤ xend，则该气球会被引爆。可以射出的弓箭的数量没有限制。 弓箭一旦被射出之后，可以无限地前进。我们想找到使得所有气球全部被引爆，所需的弓箭的最小数量。
>
>  给你一个数组 points ，其中 points [i] = [xstart,xend] ，返回引爆所有气球所必须射出的最小弓箭数。
>
>
>  示例 1：
>
>  输入：points = [[10,16],[2,8],[1,6],[7,12]]
>  输出：2
>  解释：对于该样例，x = 6 可以射爆 [2,8],[1,6] 两个气球，以及 x = 11 射爆另外两个气球
>  示例 2：
>
>  输入：points = [[1,2],[3,4],[5,6],[7,8]]
>  输出：4
>  示例 3：
>
>  输入：points = [[1,2],[2,3],[3,4],[4,5]]
>  输出：2
>  示例 4：
>
>  输入：points = [[1,2]]
>  输出：1
>  示例 5：
>
>  输入：points = [[2,3],[2,3]]
>  输出：1
>
>
>  提示：
>
>  0 <= points.length <= 104
>  points[i].length == 2
>  -231 <= xstart < xend <= 231 - 1

```java
public int findMinArrowShots(int[][] points) {
        if (points.length == 0) {
            return 0;
        }

        Arrays.sort(points, Comparator.comparingInt(o -> o[1]));

        // 不重叠区间个数
        int count = 1;
        int end = points[0][1];
        for (int i = 1; i < points.length; i++) {
            if (points[i][0] > end) {
                count++;
                end = points[i][1];
            }
        }
        return count;
    }
```



#[1115. 交替打印FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/) 

>  

```java

```



#[30. 串联所有单词的子串](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words/) 

>  给定一个字符串 s 和一些长度相同的单词 words。找出 s 中恰好可以由 words 中所有单词串联形成的子串的起始位置。
>
>  注意子串要与 words 中的单词完全匹配，中间不能有其他字符，但不需要考虑 words 中单词串联的顺序。
>
>   
>
>  示例 1：
>
>  输入：
>    s = "barfoothefoobarman",
>    words = ["foo","bar"]
>  输出：[0,9]
>  解释：
>  从索引 0 和 9 开始的子串分别是 "barfoo" 和 "foobar" 。
>  输出的顺序不重要, [9,0] 也是有效答案。
>  示例 2：
>
>  输入：
>    s = "wordgoodgoodgoodbestword",
>    words = ["word","good","best","word"]
>  输出：[]

```java
public List<Integer> findSubstring(String s, String[] words) {
           int left, right, len;

        // 所有单词数
        int size = words.length;

        List<Integer> res = new ArrayList<>(10);

        // 如果目标数组为空，则返回一个空集合
        if (words.length == 0) {
            return res;
        } else {
            // 单词长度
            len = words[0].length();
        }

        // 定义两个map集合，一个存目标单词，一个存滑动窗口
        Map<String, Integer> needs = new HashMap<>(5);
        Map<String, Integer> windows = new HashMap<>(10);

        // 初始化目标集合，将单词与出现的次数对应存入map集合中
        for (String word : words) {
            // 如果单词存在集合中，则返回出现的次数，否则返回0
            int count = needs.getOrDefault(word, 0);
            // 存入map中，次数+1
            needs.put(word, count + 1);
        }

        // 单词的匹配数量（包括单词和出现次数）
        int match;

        // 由于字符串不一定是单词长度的倍数，所以需要遍历一个单词长度的所有情况
        for (int i = 0; i < len; i++) {
            // 初始化左右指针开始处为i，match初始化为0
            right = left = i;
            match = 0;

            // 右指针最多到字符串的最后一个单词开始位置
            while (right <= s.length() - len) {

                // 向右滑动，存入单词和出现的次数
                String s1 = s.substring(right, right + len);
                // 以单词长度为步长移动右指针
                right += len;
                int count = windows.getOrDefault(s1, 0);
                windows.put(s1, count + 1);

                // 如果单词和出现的次数与目标一致，则匹配+1
                if (needs.containsKey(s1) && windows.get(s1).intValue() == needs.get(s1).intValue()) {
                    match++;
                }

                // 当匹配数等于目标集合的大小（说明已经覆盖了目标集合）
                while (left < right && match == needs.size()) {

                    // right - left / len求出窗口中单词数，如果等于目标单词数，则匹配成功，将左指针位置加入list
                    if ((right - left) / len == size) {
                        res.add(left);
                    }

                    // 左指针右移，类似右指针方法
                    String s2 = s.substring(left, left + len);
                    left += len;
                    windows.put(s2, windows.get(s2) - 1);

                    if (needs.containsKey(s2) && windows.get(s2) < needs.get(s2)) {
                        match--;
                    }
                }
            }
            // 清空窗口，进行下一次遍历
            windows.clear();
        }

        return res;
    }
```



#[567. 字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/) 

>  输入一个字符串，打印出该字符串中字符的所有排列。
>
>   
>
>  你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。
>
>   
>
>  示例:
>
>  输入：s = "abc"
>  输出：["abc","acb","bac","bca","cab","cba"]
>
>
>  限制：
>
>  1 <= s 的长度 <= 8

```java
 private List<String> list = new ArrayList<>();

    public String[] permutation(String s) {
        perm(s.toCharArray(), 0, s.length() - 1);
        
        return list.toArray(new String[0]);
    }

    private void perm(char[] seq, int curPos, int n) {
        if (curPos == n) {
            list.add(new String(seq));
        } else {
            for (int i = curPos; i <= n; i++) {
                // 判断第i个元素是否在seq[curPos,i－1]中出现过,如果出现过就不用交换了
                if(!findSame(seq,curPos,i)){
                    swap(seq, curPos, i);
                    perm(seq, curPos + 1, n);
                    swap(seq, i, curPos);
                }
            }
        }
    }

    private boolean findSame(char[] seq, int from, int candidate) {
        for (int j = from; j < candidate; j++) {
            if(seq[j] == seq[candidate]){
                return true;
            }
        }
        return false;
    }

    private void swap(char[] seq, int i, int j) {
        char tmp = seq[i];
        seq[i] = seq[j];
        seq[j] = tmp;
    }
```



#[63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/) 

>  一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
>
>  机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
>
>  现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？
>

```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if (obstacleGrid == null || obstacleGrid[0].length == 0) {
            return 0;
        }
        int rol = obstacleGrid.length;
        int col = obstacleGrid[0].length;
        for (int i = 0; i < rol; i++) {  //这里表示行下的某列
            for (int j = 0; j < col; j++) {
                if(obstacleGrid[i][j]==1){
                    obstacleGrid[i][j]=0;
                    continue;
                }
                if(i==0&&j==0){
                    obstacleGrid[i][j]=1;
                }else if(i==0){
                    obstacleGrid[i][j]=obstacleGrid[i][j-1];
                }else if(j==0){
                    obstacleGrid[i][j]=obstacleGrid[i-1][j];
                }else{
                    obstacleGrid[i][j]=obstacleGrid[i-1][j]+obstacleGrid[i][j-1];
                }
            }
        } 
        return obstacleGrid[rol-1][col-1];  
    }
```



#[409. 最长回文串](https://leetcode-cn.com/problems/longest-palindrome/) 

>  
>  给定一个包含大写字母和小写字母的字符串，找到通过这些字母构造成的最长的回文串。
>
>  在构造过程中，请注意区分大小写。比如 `"Aa"` 不能当做一个回文字符串。
>
>  **注意:**
>  假设字符串的长度不会超过 1010。
>
>  **示例 1:**
>
>  ```java
>  输入:
>  "abccccdd"
>  
>  输出:
>  7
>  
>  解释:
>  我们可以构造的最长的回文串是"dccaccd", 它的长度是 7。
>  ```

```java
 public int longestPalindrome(String s) {
        int[] count = new int[128];
        for (char c: s.toCharArray())
            count[c]++;

        int ans = 0;
        for (int v: count) {
            ans += v / 2 * 2;
            if (v % 2 == 1 && ans % 2 == 0)
                ans++;
        }
        return ans;
    }
```



#[735. 行星碰撞](https://leetcode-cn.com/problems/asteroid-collision/) 

>  给定一个整数数组 asteroids，表示在同一行的行星。
>
>  对于数组中的每一个元素，其绝对值表示行星的大小，正负表示行星的移动方向（正表示向右移动，负表示向左移动）。每一颗行星以相同的速度移动。
>
>  找出碰撞后剩下的所有行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。
>
>  示例 1:
>
>  输入: 
>  asteroids = [5, 10, -5]
>  输出: [5, 10]
>  解释: 
>  10 和 -5 碰撞后只剩下 10。 5 和 10 永远不会发生碰撞。
>  示例 2:
>
>  输入: 
>  asteroids = [8, -8]
>  输出: []
>  解释: 
>  8 和 -8 碰撞后，两者都发生爆炸。
>  示例 3:
>
>  输入: 
>  asteroids = [10, 2, -5]
>  输出: [10]
>  解释: 
>  2 和 -5 发生碰撞后剩下 -5。10 和 -5 发生碰撞后剩下 10。
>  示例 4:
>
>  输入: 
>  asteroids = [-2, -1, 1, 2]
>  输出: [-2, -1, 1, 2]
>  解释: 
>  -2 和 -1 向左移动，而 1 和 2 向右移动。
>  由于移动方向相同的行星不会发生碰撞，所以最终没有行星发生碰撞。
>  说明:
>
>  数组 asteroids 的长度不超过 10000。
>  每一颗行星的大小都是非零整数，范围是 [-1000, 1000] 。

```java
  public int[] asteroidCollision(int[] asteroids) {
        int[] nums=new int[asteroids.length];
        int x = 0;
        for(int i = 0;i<asteroids.length;i++){
            if(asteroids[i]<0 && x>0 && nums[x-1]>0){//判断行星方向是否与上一颗行星方向冲突
                if(nums[x-1]+asteroids[i]<0){//冲突的行星中向左的更大，回溯上一颗，保留本颗大的行星
                    x=x-1;
                    i--;
                }else if(nums[x-1]+asteroids[i]==0){ //两颗一样大时，去掉行星
                    x=x-1;
                }
            }else{//无冲突时直接添加
                nums[x]=asteroids[i];
                x=x+1;
            }
        }
        int[] res = new int[x];//将行星拿出来重新构建数组
        for(int i=0;i<x;i++){
            res[i]=nums[i];
        }
        return res;
    }
```



# [264. 丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/)

>  编写一个程序，找出第 n 个丑数。
>
>  丑数就是质因数只包含 2, 3, 5 的正整数。
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



