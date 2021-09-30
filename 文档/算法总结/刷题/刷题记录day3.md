[toc]

# [135. 分发糖果](https://leetcode-cn.com/problems/candy/)

>   老师想给孩子们分发糖果，有 N 个孩子站成了一条直线，老师会根据每个孩子的表现，预先给他们评分。
>
>   你需要按照以下要求，帮助老师给这些孩子分发糖果：
>
>   每个孩子至少分配到 1 个糖果。
>   相邻的孩子中，评分高的孩子必须获得更多的糖果。
>   那么这样下来，老师至少需要准备多少颗糖果呢？
>
>   示例 1:
>
>   输入: [1,0,2]
>   输出: 5
>   解释: 你可以分别给这三个孩子分发 2、1、2 颗糖果。
>   示例 2:
>
>   输入: [1,2,2]
>   输出: 4
>   解释: 你可以分别给这三个孩子分发 1、2、1 颗糖果。
>        第三个孩子只得到 1 颗糖果，这已满足上述两个条件。
>

```java
  public int candy(int[] ratings) {
        if (ratings == null || ratings.length == 0) {
            return -1;
        }
        int[] L = new int[ratings.length];
        for (int i = 0; i < ratings.length; i++) {
            L[i] = 1;
        }
        for (int i = 1; i < ratings.length; i++) {
            if (ratings[i] > ratings[i - 1] && L[i] <= L[i - 1]) {
                L[i] = L[i - 1] + 1;
            }
        }
        for (int i = ratings.length - 2; i >= 0; i--) {
            if (ratings[i] > ratings[i + 1] && L[i] <= L[i + 1]) {
                L[i] = L[i + 1] + 1;
            }
        }
        int sum = 0;
        for (int i = 0; i < L.length; i++) {
            sum += L[i];
        }
        return sum;
    }
```

# [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/) 经典dp

>   给你一个字符串 s 和一个字符规律 p，请你来实现一个支持 '.' 和 '*' 的正则表达式匹配。
>
>   '.' 匹配任意单个字符
>   '*' 匹配零个或多个前面的那一个元素
>   所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串。
>
>
>   示例 1：
>
>   输入：s = "aa" p = "a"
>   输出：false
>   解释："a" 无法匹配 "aa" 整个字符串。
>   示例 2:
>
>   输入：s = "aa" p = "a*"
>   输出：true
>   解释：因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。
>   示例 3：
>
>   输入：s = "ab" p = ".*"
>   输出：true
>   解释：".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。
>   示例 4：
>
>   输入：s = "aab" p = "c*a*b"
>   输出：true
>   解释：因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。
>   示例 5：
>
>   输入：s = "mississippi" p = "mis*is*p*."
>   输出：false
>
>
>   提示：
>
>   0 <= s.length <= 20
>   0 <= p.length <= 30
>   s 可能为空，且只包含从 a-z 的小写字母。
>   p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 *。
>   保证每次出现字符 * 时，前面都匹配到有效的字符

```java
 public boolean isMatch(String s, String p) {
    if (s == null || p == null)
			return false;
		int m = s.length(), n = p.length();
		boolean[][] res = new boolean[m + 1][n + 1];
		res[0][0] = true;
		//s.charAt(i)与p.charAt(j)的结果是存储在res[i+1][j+1]中
		//所以res[0][i]其实是存储p中与0个字符(s.charAt(-1)不存在)的匹配结果
		for (int i = 0; i < n; i++) {
			if (p.charAt(i) == '*' && res[0][i - 1])
				res[0][i + 1] = true;
		}
		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (p.charAt(j) == '.')
					res[i + 1][j + 1] = res[i][j];
				if (p.charAt(j) == s.charAt(i))
					res[i + 1][j + 1] = res[i][j];
				if (p.charAt(j) == '*') {
					if (s.charAt(i) != p.charAt(j - 1) && p.charAt(j - 1) != '.')
						res[i + 1][j + 1] = res[i + 1][j - 1];
					else {
						//res[i + 1][j - 1] 表示*一个都不匹配;
						//res[i + 1][j]表示匹配一个 
						//res[i][j + 1]表示匹配多个
						res[i + 1][j + 1] = res[i + 1][j - 1] || res[i + 1][j] || res[i][j + 1];
					}
				}
			}
		}
		return res[m][n];
    }
```

# [9. 回文数](https://leetcode-cn.com/problems/palindrome-number/)

>   判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。
>
>   示例 1:
>
>   输入: 121
>   输出: true
>   示例 2:
>
>   输入: -121
>   输出: false
>   解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
>   示例 3:
>
>   输入: 10
>   输出: false
>   解释: 从右向左读, 为 01 。因此它不是一个回文数。
>   进阶:
>
>   你能不将整数转为字符串来解决这个问题吗？
>

```java
public boolean isPalindrome(int x) {
        if(x<0) return false;
        int temp=x;
        int y=0;
        while(x>0){
            y*=10;
            y+=x%10;
            x/=10;
        }
        return temp==y;
    }
```

# [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

>   给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。
>
>   你可以认为每种硬币的数量是无限的。
>
>    
>
>   示例 1：
>
>   输入：coins = [1, 2, 5], amount = 11
>   输出：3 
>   解释：11 = 5 + 5 + 1
>   示例 2：
>
>   输入：coins = [2], amount = 3
>   输出：-1
>   示例 3：
>
>   输入：coins = [1], amount = 0
>   输出：0
>   示例 4：
>
>   输入：coins = [1], amount = 1
>   输出：1
>   示例 5：
>
>   输入：coins = [1], amount = 2
>   输出：2
>
>
>   提示：
>
>   1 <= coins.length <= 12
>   1 <= coins[i] <= 231 - 1
>   0 <= amount <= 104

```java
 public int coinChange(int[] coins, int amount) {
    int max = amount + 1;
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, max);
    dp[0] = 0;
    for (int i = 1; i <= amount; i++) {
      for (int j = 0; j < coins.length; j++) {
        if (coins[j] <= i) {
          dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
        }
      }
    }
    return dp[amount] > amount ? -1 : dp[amount];
 }
```

# [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

>   做跟右

```java
 public List<Integer> inorderTraversal(TreeNode root) {
        Stack<TreeNode> stack = new Stack<>();
        TreeNode cur = root;
        List<Integer> res = new ArrayList<>();
        while(cur != null || !stack.isEmpty()){
            while(cur != null){
                stack.push(cur);
                cur = cur.left;
            }
            cur = stack.pop();
            res.add(cur.val);
            cur = cur.right;
        }
        return res;
    }
```

# [41. 缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

>
>   给你一个未排序的整数数组，请你找出其中没有出现的最小的正整数。
>
>    
>
>   **示例 1:**
>
>   ```
>   输入: [1,2,0]
>   输出: 3
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: [3,4,-1,1]
>   输出: 2
>   ```
>
>   **示例 3:**
>
>   ```
>   输入: [7,8,9,11,12]
>   输出: 1 
>   ```
>
>   **提示：**
>
>   你的算法的时间复杂度应为O(*n*)，并且只能使用常数级别的额外空间。

```java
 public int firstMissingPositive(int[] nums) {
           int n = nums.length;

        // 基本情况
        int contains = 0;
        for (int i = 0; i < n; i++)
            if (nums[i] == 1) {
                contains++;
                break;
            }

        if (contains == 0)
            return 1;

        // nums = [1]
        if (n == 1)
            return 2;

        // 用 1 替换负数，0，
        // 和大于 n 的数
        // 在转换以后，nums 只会包含
        // 正数
        for (int i = 0; i < n; i++)
            if ((nums[i] <= 0) || (nums[i] > n))
                nums[i] = 1;

        // 使用索引和数字符号作为检查器
        // 例如，如果 nums[1] 是负数表示在数组中出现了数字 `1`
        // 如果 nums[2] 是正数 表示数字 2 没有出现
        for (int i = 0; i < n; i++) {
            int a = Math.abs(nums[i]);
            // 如果发现了一个数字 a - 改变第 a 个元素的符号
            // 注意重复元素只需操作一次
            if (a == n)
                nums[0] = -Math.abs(nums[0]);
            else
                nums[a] = -Math.abs(nums[a]);
        }

        // 现在第一个正数的下标
        // 就是第一个缺失的数
        for (int i = 1; i < n; i++) {
            if (nums[i] > 0)
                return i;
        }

        if (nums[0] > 0)
            return n;

        return n + 1;
    }
```



# [300. 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/) 

>   给定一个无序的整数数组，找到其中最长上升子序列的长度。
>
>   示例:
>
>   输入: [10,9,2,5,3,7,101,18]
>   输出: 4 
>   解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
>   说明:
>
>   可能会有多种最长上升子序列的组合，你只需要输出对应的长度即可。
>   你算法的时间复杂度应该为 O(n2) 。
>   进阶: 你能将算法的时间复杂度降低到 O(n log n) 吗?

#### 方法一：动态规划

```java
 public int lengthOfLIS(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int[] dp = new int[nums.length];
        dp[0] = 1;
        int maxans = 1;
        for (int i = 1; i < nums.length; i++) {
            dp[i] = 1;
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            maxans = Math.max(maxans, dp[i]);
        }
        return maxans;
    }
```

#### 方法二：贪心 + 二分查找

```java
 public int lengthOfLIS(int[] nums) {
        int[] tails = new int[nums.length];
        int res = 0;
        for(int num : nums) {
            int i = 0, j = res;
            while(i < j) {
                int m = (i + j) / 2;
                if(tails[m] < num) i = m + 1;
                else j = m;
            }
            tails[i] = num;
            if(res == j) res++;
        }
        return res;
    }
```

# [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/) 快学双指针,请闭着眼睛写

>   给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。
>
>    
>
>   说明：
>
>   初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
>   你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。
>
>
>   示例：
>
>   输入：
>   nums1 = [1,2,3,0,0,0], m = 3
>   nums2 = [2,5,6],       n = 3
>
>   输出：[1,2,2,3,5,6]
>
>
>   提示：
>
>   -10^9 <= nums1[i], nums2[i] <= 10^9
>   nums1.length == m + n
>   nums2.length == n

```java
 //合并两个有序数组
    //nums1 = [1,2,3,0,0,0], m = 3
    //nums2 = [2,5,6],       n = 3
    public void merge(int[] nums1, int m, int[] nums2, int n) {
      	//
        int p1 = m - 1;
        int p2 = n - 1;
        int p = m + n - 1;
        while ((p1 >= 0) && (p2 >= 0)) {
            nums1[p--] = (nums1[p1] < nums2[p2]) ? nums2[p2--] : nums1[p1--];
        }
        System.arraycopy(nums2, 0, nums1, 0, p2 + 1);
    }
```

# [198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/) 快学dp

>   你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
>   给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。
>
>    
>
>   示例 1：
>
>   输入：[1,2,3,1]
>   输出：4
>   解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
>        偷窃到的最高金额 = 1 + 3 = 4 。
>   示例 2：
>
>   输入：[2,7,9,3,1]
>   输出：12
>   解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
>        偷窃到的最高金额 = 2 + 9 + 1 = 12 。
>
>
>   提示：
>
>   0 <= nums.length <= 100
>   0 <= nums[i] <= 400

```java
 public int rob(int[] nums) {
        if(nums == null||nums.length == 0)return 0;
        if(nums.length == 1){
            return nums[0];
        }
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0],nums[1]);
        for(int i = 2;i<nums.length;i++){
            dp[i] = Math.max(nums[i]+dp[i-2],dp[i-1]);
        }
        return dp[nums.length-1];
    }
```

# [79. 单词搜索](https://leetcode-cn.com/problems/word-search/) 经典

>   给定一个二维网格和一个单词，找出该单词是否存在于网格中。
>
>   单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。
>
>    
>
>   示例:
>
>   board =
>   [
>     ['A','B','C','E'],
>     ['S','F','C','S'],
>     ['A','D','E','E']
>   ]
>
>   给定 word = "ABCCED", 返回 true
>   给定 word = "SEE", 返回 true
>   给定 word = "ABCB", 返回 false
>
>
>   提示：
>
>   board 和 word 中只包含大写和小写英文字母。
>   1 <= board.length <= 200
>   1 <= board[i].length <= 200
>   1 <= word.length <= 10^3

```java
public boolean exist(char[][] board, String word) {
         for (int i = 0; i < board.length; i++){
            for (int j = 0; j < board[0].length; j++) {
                if (search(board, word, i, j, 0)) {
                    return true;
                }
            }
        }
        return  false;
    }
       boolean search(char[][] board, String word, int i, int j, int k) {
        if (k >= word.length()) return true;
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length || board[i][j] != word.charAt(k)) return false;
        board[i][j] += 256;
        boolean result = search(board, word, i - 1, j, k + 1) || search(board, word, i + 1, j, k + 1)
                || search(board, word, i, j - 1, k + 1) || search(board, word, i, j + 1, k + 1);
        board[i][j] -= 256;
        return result;
    }
```

# [78. 子集](https://leetcode-cn.com/problems/subsets/) 经典回溯

>   给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。
>
>   说明：解集不能包含重复的子集。
>
>   示例:
>
>   输入: nums = [1,2,3]
>   输出:
>   [
>     [3],
>     [1],
>     [2],
>     [1,2,3],
>     [1,3],
>     [2,3],
>     [1,2],
>     []
>   ]

```java
 public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        dfs(nums, res, 0, new ArrayList<>());
        return res;
    }

    private void dfs(int[] nums, List<List<Integer>> res, int cur, List<Integer> list) {
        res.add(new ArrayList<>(list));
				//
        for (int i = cur; i < nums.length; ++i) {
            list.add(nums[i]);
            //i+1和cur+1的区别
            dfs(nums, res, i + 1, list);
            list.remove(list.size() - 1);
        }
    }
```

# [14. 最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix/) 其实没你想的复杂

>   编写一个函数来查找字符串数组中的最长公共前缀。
>
>   如果不存在公共前缀，返回空字符串 ""。
>
>   示例 1:
>
>   输入: ["flower","flow","flight"]
>   输出: "fl"
>   示例 2:
>
>   输入: ["dog","racecar","car"]
>   输出: ""
>   解释: 输入不存在公共前缀。
>   说明:
>
>   所有输入只包含小写字母 a-z 。
>

```java
  public String longestCommonPrefix(String[] strs) {
        if(strs.length==0) return "";
        String prefix=strs[0];
        for(int i=1;i<strs.length;i++){
            while(strs[i].indexOf(prefix)!=0){
                prefix=prefix.substring(0,prefix.length()-1);
                if(prefix.isEmpty()) return "";
            }
        }
        return prefix;
    }
```

# [112. 路径总和](https://leetcode-cn.com/problems/path-sum/) 其实我更简单

>   给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。
>
>   说明: 叶子节点是指没有子节点的节点。
>
>   示例: 
>   给定如下二叉树，以及目标和 sum = 22，
>
>                 5
>                / \
>               4   8
>              /   / \
>             11  13  4
>            /  \      \
>           7    2      1
>   返回 true, 因为存在目标和为 22 的根节点到叶子节点的路径 5->4->11->2。
>

```java
 public boolean hasPathSum(TreeNode root, int sum) {
        if (root == null) return false;
        sum = sum - root.val;
        if (sum == 0 && root.left == null && root.right == null) {
            return true;
        }
        return hasPathSum(root.left, sum) || hasPathSum(root.right, sum);
    }
```

# [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/) 快学快慢指针

>   

```java
 public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while(fast != null && fast.next != null){
            fast = fast.next.next;
            slow = slow.next;
            if(fast == slow){
                return true;
            }
        }
        return false;
    }
```

# [160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

>   

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
             /**
        定义两个指针, 第一轮让两个到达末尾的节点指向另一个链表的头部, 最后如果相遇则为交点(在第一轮移动中恰好抹除了长度差)
        两个指针等于移动了相同的距离, 有交点就返回, 无交点就是各走了两条指针的长度
        **/
        if(headA == null || headB == null) return null;
        ListNode pA = headA, pB = headB;
        // 在这里第一轮体现在pA和pB第一次到达尾部会移向另一链表的表头, 而第二轮体现在如果pA或pB相交就返回交点, 不相交最后就是null==null
        while(pA != pB) {
            pA = pA == null ? headB : pA.next;
            pB = pB == null ? headA : pB.next;
        }
        return pA;
    }
```

# [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/) 不想写sql

>   

```java

```

# [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/) so easy

>   给定一个二叉树，检查它是否是镜像对称的。
>
>    
>
>   例如，二叉树 [1,2,2,3,4,4,3] 是对称的。
>
>       1
>      / \
>     2   2
>    / \ / \
>   3  4 4  3
>
>
>   但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:
>
>       1
>      / \
>     2   2
>      \   \
>      3    3

```java
  public boolean isMirror(TreeNode root1,TreeNode root2){
        if(root1==null && root2==null)
            return true;
        else if(root1==null || root2==null)
            return false;
        boolean left = isMirror(root1.left,root2.right);
        boolean right = isMirror(root1.right,root2.left);
        return root1.val == root2.val && left && right;
    }
    public boolean isSymmetric(TreeNode root) {
        return isMirror(root,root);
    }
```

# [543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/) so easy

>   给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。
>
>    
>
>   示例 :
>   给定二叉树
>
>             1
>            / \
>           2   3
>          / \     
>         4   5    
>   返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。
>

```java
 int ans;
    public int diameterOfBinaryTree(TreeNode root) {
        ans = 1;
        depth(root);
        return ans - 1;
    }

    public int depth(TreeNode node) {
        // 访问到空节点了，返回0
        if (node == null) return 0; 
        // 左儿子为根的子树的深度
        int L = depth(node.left); 
         // 右儿子为根的子树的深度
        int R = depth(node.right);
         // 计算d_node即L+R+1 并更新ans
        ans = Math.max(ans, L + R + 1);
        // 返回该节点为根的子树的深度
        return Math.max(L, R) + 1; 
    }
```

# [236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/) 经典的简单题,透彻心扉哈哈哈

>   

```java
 public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
   			//返回root的情况
        if(root == null || root == p || root == q){
            return root;
        }
   			//查找左边
        TreeNode left = lowestCommonAncestor(root.left,p,q);
   			//查找右边
        TreeNode right = lowestCommonAncestor(root.right,p,q);
   			//如果都不为null,返回root
        if(left != null && right != null ){
            return root;
        }
   			//返回非空的那个
        return left == null ? right : left;
    }
```

# [175. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/) 有被冒犯到

>   表1: Person
>
>   +-------------+---------+
>   | 列名         | 类型     |
>   +-------------+---------+
>   | PersonId    | int     |
>   | FirstName   | varchar |
>   | LastName    | varchar |
>   +-------------+---------+
>   PersonId 是上表主键
>   表2: Address
>
>   +-------------+---------+
>   | 列名         | 类型    |
>   +-------------+---------+
>   | AddressId   | int     |
>   | PersonId    | int     |
>   | City        | varchar |
>   | State       | varchar |
>   +-------------+---------+
>   AddressId 是上表主键

```sql
#编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：
#FirstName, LastName, City, State
SELECT Person.FirstName,Person.LastName,Address.City,Address.State
From Person left join Address
on Person.PersonId=Address.PersonId
```

# [204. 计数质数](https://leetcode-cn.com/problems/count-primes/)

> 统计所有小于非负整数 n 的质数的数量。
>
>  
>
> 示例 1：
>
> 输入：n = 10
> 输出：4
> 解释：小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。
> 示例 2：
>
> 输入：n = 0
> 输出：0
> 示例 3：
>
> 输入：n = 1
> 输出：0
>
>
> 提示：
>
> 0 <= n <= 5 * 106

```java

```

