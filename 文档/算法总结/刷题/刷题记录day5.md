[toc]

# [96. 不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

>   给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？
>
>   示例:
>
>   输入: 3
>   输出: 5
>   解释:
>   给定 n = 3, 一共有 5 种不同结构的二叉搜索树:
>
>      1         3     3      2      1
>       \       /     /      / \      \
>        3     2     1      1   3      2
>       /     /       \                 \
>      2     1         2                 3
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/unique-binary-search-trees
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
 public int numTrees(int n) {
        // 提示：我们在这里需要用 long 类型防止计算过程中的溢出
        long C = 1;
        for (int i = 0; i < n; ++i) {
            C = C * 2 * (2 * i + 1) / (i + 2);
        }
        return (int) C;
    }
```

# [415. 字符串相加](https://leetcode-cn.com/problems/add-strings/)

>   给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和。
>
>    提示：
>
>   num1 和num2 的长度都小于 5100
>num1 和num2 都只包含数字 0-9
>   num1 和num2 都不包含任何前导零
>   你不能使用任何內建 BigInteger 库， 也不能直接将输入的字符串转换为整数形式

```java
 public String addStrings(String num1, String num2) {
   StringBuilder sb = new StringBuilder();
        int carry = 0;
        for (int i = num1.length() - 1, j = num2.length() - 1; i >= 0 || j >= 0 || carry == 1; i--, j--) {
            int x = i < 0 ? 0 : num1.charAt(i) - '0';
            int y = j < 0 ? 0 : num2.charAt(j) - '0';
            sb.append((x + y + carry) % 10);
            carry = (x + y + carry) /10;
        }
        return sb.reverse().toString();
    }
```

# [43. 字符串相乘](https://leetcode-cn.com/problems/multiply-strings) 

> 给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。
>
> 示例 1:
>
> 输入: num1 = "2", num2 = "3"
> 输出: "6"
> 示例 2:
>
> 输入: num1 = "123", num2 = "456"
> 输出: "56088"
> 说明：
>
> num1 和 num2 的长度小于110。
> num1 和 num2 只包含数字 0-9。
> num1 和 num2 均不以零开头，除非是数字 0 本身。
> 不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。

```java
 public String multiply(String num1, String num2) {
        int sumLen = num1.length() + num2.length();
        int[] res = new int[sumLen];
        for (int i = 0; i < num1.length(); i++) {
            int num11 = num1.charAt(num1.length() - 1 - i) - '0';//3
            for (int j = 0; j < num2.length(); j++) {
                int num22 = num2.charAt(num2.length() - 1 - j) - '0';//6,5,4
                res[i + j] += num11 * num22;            //序列和相同相加
            }
        }
        for (int i = 0; i < res.length - 1; i++) {
            if (res[i] >= 10) {
                res[i + 1] += res[i] / 10;//后位加上
                res[i] %= 10;//余数
            }
        }
        int i = res.length - 1;
        while (i > 0 && res[i] == 0) {
            i--;
        } // 去除结果前面的 0
        StringBuilder sb = new StringBuilder();
        for (; i >= 0; i--) {
            sb.append(res[i]);
        }
        return sb.toString();
    }
```



# [309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

>   给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。
>
>   设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
>
>   你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>   卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
>   示例:
>
>   输入: [1,2,3,0,2]
>   输出: 3 
>   解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]

```java
public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) {
            return 0;
        }
        //由于可以无限次交易，所以只定义两个维度，第一个维度是天数，
  			//第二个维度表示是否持有股票，0表示不持有，1表示持有，2表示过渡期
        int[][] dp = new int[prices.length][3];
        dp[0][0] = 0;
        dp[0][1] = -prices[0];
        dp[0][2] = 0;
        for (int i = 1; i < prices.length; i++) {
            //第i天不持有股票的情况有两种
            //a.第i-1天也不持有股票
            //b.第i-1天是过渡期
            dp[i][0] = Math.max(dp[i-1][0], dp[i-1][2]);
            //第i天持有股票有两种情况
            //a.第i-1天也持有股票，第i天不操作，
            //b.第i-1天不持有股票，在第i天买入
            dp[i][1] = Math.max(dp[i-1][1], dp[i-1][0] - prices[i]);
            //第i天是冷冻期只有一种情况，第i-1天持有股票且卖出
            dp[i][2] = dp[i-1][1] + prices[i];
        }
        //最后最大利润为最后一天，不持有股票或者进入冷冻期的情况
        return Math.max(dp[prices.length-1][0], dp[prices.length-1][2]);
    }
```

# [剑指 Offer 42 - 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

>   输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。
>
>   要求时间复杂度为O(n)。
>
>   **示例1:**
>
>   ```
>输入: nums = [-2,1,-3,4,-1,2,1,-5,4]
>    输出: 6
>解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
>   ```

```java
  public int maxSubArray(int[] nums) {
        if (nums.length == 0) return 0;
        int res = Integer.MIN_VALUE;
        int maxsum = 0;
        for (int num : nums) {
            maxsum = Math.max(0, maxsum) + num;
            res = Math.max(res, maxsum);
        }
        return res;
    }
```

# [71. 简化路径](https://leetcode-cn.com/problems/simplify-path/)

>   以 Unix 风格给出一个文件的绝对路径，你需要简化它。或者换句话说，将其转换为规范路径。
>
>   在 Unix 风格的文件系统中，一个点（.）表示当前目录本身；此外，两个点 （..） 表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。更多信息请参阅：Linux / Unix中的绝对路径 vs 相对路径
>
>   请注意，返回的规范路径必须始终以斜杠 / 开头，并且两个目录名之间必须只有一个斜杠 /。最后一个目录名（如果存在）不能以 / 结尾。此外，规范路径必须是表示绝对路径的最短字符串。
>
>   示例 1：
>
>   输入："/home/"
>   输出："/home"
>   解释：注意，最后一个目录名后面没有斜杠。
>   示例 2：
>
>   输入："/../"
>   输出："/"
>   解释：从根目录向上一级是不可行的，因为根是你可以到达的最高级。
>   示例 3：
>
>   输入："/home//foo/"
>   输出："/home/foo"
>   解释：在规范路径中，多个连续斜杠需要用一个斜杠替换。

```java
public String simplifyPath(String path) {
        // 利用栈或队列来解决
        LinkedList<String> queue = new LinkedList<>();
        String[] paths = path.split("/");
        for (String p : paths) {
            if ("".equals(p) || ".".equals(p)) {
                continue;
            } else if ("..".equals(p)) {
                if (!queue.isEmpty()) {
                    queue.removeLast();
                }
            } else {
                queue.addLast("/" + p);
            }
        }
        if (queue.isEmpty()) {
            return "/";
        } else {
            StringBuilder res = new StringBuilder();
            while (!queue.isEmpty()) {
                res.append(queue.removeFirst());
            }
            return res.toString();
        }
    }
```

# [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

>   给定一个二叉树和一个目标和，找到所有从根节点到叶子节点路径总和等于给定目标和的路径。
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
>            /  \    / \
>           7    2  5   1
>   返回:
>
>   [
>      [5,4,11,2],
>      [5,8,4,5]
>   ]

```java
public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> paths = new ArrayList<>();
        findPaths(root,sum,paths,new ArrayList<>());
        return paths;
    }
    private void findPaths(TreeNode node,int sum,List<List<Integer>> paths,List<Integer> current){
        if(node == null){
            return;
        }
        current.add(node.val);
        if(node.val == sum && node.left == null && node.right == null){
            //深拷贝
            paths.add(new ArrayList<>(current));
        }else{
         findPaths(node.left,sum - node.val,paths,current);
         findPaths(node.right,sum - node.val,paths,current);
        }
         current.remove(current.size() - 1);
    }
```

# [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

>   给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。
>
>   candidates 中的数字可以无限制重复被选取。
>
>   说明：
>
>   所有数字（包括 target）都是正整数。
>   解集不能包含重复的组合。 
>   示例 1：
>
>   输入：candidates = [2,3,6,7], target = 7,
>   所求解集为：
>   [
>     [7],
>     [2,2,3]
>   ]
>   示例 2：
>
>   输入：candidates = [2,3,5], target = 8,
>   所求解集为：
>   [
>     [2,2,2,2],
>     [2,3,3],
>     [3,5]
>   ]
>

```java
   public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> res = new ArrayList<>();
        //答案,源,目标,begin,剩余,子答案
        help(res,candidates,target,0,candidates.length,new ArrayList<>());
        return res;
    }
    private void help(List<List<Integer>> res,int[] candidates,
                      int target,int begin,int len,List<Integer> list){
        if(target == 0){res.add(new ArrayList<>(list));}
        for(int i = begin;i<len;i++){
            if(target - candidates[i] < 0) return;
            list.add(candidates[i]);
            help(res,candidates,target-candidates[i],i,len,list);
            list.remove(list.size()-1);
        }
    }
```

# [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

>   给定一个非负整数数组，你最初位于数组的第一个位置。
>
>   数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
>   判断你是否能够到达最后一个位置。
>
>   示例 1:
>
>   输入: [2,3,1,1,4]
>   输出: true
>   解释: 我们可以先跳 1 步，从位置 0 到达 位置 1, 然后再从位置 1 跳 3 步到达最后一个位置。
>   示例 2:
>
>   输入: [3,2,1,0,4]
>   输出: false
>   解释: 无论怎样，你总会到达索引为 3 的位置。但该位置的最大跳跃长度是 0 ， 所以你永远不可能到达最后一个位置。
>

```java
   public boolean canJump(int[] nums) {
        int lastPos = nums.length - 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            if (i + nums[i] >= lastPos) {
                lastPos = i;
            }
        }
        return lastPos == 0;
    }
```

# [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

>   给你二叉树的根节点 `root` ，返回它节点值的 **前序** 遍历。

```java
   public List<Integer> preorderTraversal(TreeNode root) {
        //迭代
        List<Integer> res = new ArrayList<>();
        if(root == null){
            return res;
        }
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        while(!stack.isEmpty()){
            TreeNode s = stack.pop();
            res.add(s.val);
            if(s.right != null){
                stack.push(s.right);
            }
            if(s.left != null){
                stack.push(s.left);
            }
        }
        return res;
    }
```

# [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

>   请判断一个链表是否为回文链表。
>
>   **示例 1:**
>
>   ```
>   输入: 1->2
>   输出: false
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: 1->2->2->1
>   输出: true
>   ```

```java
 public boolean isPalindrome(ListNode head) {
        	 ListNode slow=head;
		 ListNode fast=head;
		 
		 if(fast==null||fast.next==null)//0个节点或是1个节点
			 return true;
 
 
		 while(fast.next!=null&&fast.next.next!=null)
		 {
			 fast=fast.next.next;
			 slow=slow.next;
		 }
		 //对链表后半段进行反转
		 ListNode midNode=slow;
		 ListNode firNode=slow.next;//后半段链表的第一个节点
		 ListNode cur=firNode.next;//插入节点从第一个节点后面一个开始
		 firNode.next=null;//第一个节点最后会变最后一个节点
		 while(cur!=null)
		 {
			 ListNode nextNode=cur.next;//保存下次遍历的节点
			 cur.next=midNode.next;
			 midNode.next=cur;
			 cur=nextNode;
		 }
		 
		 //反转之后对前后半段进行比较
		 slow=head;
		 fast=midNode.next;
		 while(fast!=null)
		 {
			 if(fast.val!=slow.val)
				 return false;
			 slow=slow.next;
			 fast=fast.next;
		 }
		 return true;
    }
```

# [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

>   翻转一棵二叉树。
>
>   示例：
>
>   输入：
>
>        4
>      /   \
>     2     7
>    / \   / \
>   1   3 6   9
>   输出：
>
>        4
>      /   \
>     7     2
>    / \   / \
>   9   6 3   1

```java
  public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        TreeNode right = invertTree(root.right);
        TreeNode left = invertTree(root.left);
        root.left = right;
        root.right = left;
        return root;
    }
```

# [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

>   给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
>
>   你的算法时间复杂度必须是 O(log n) 级别。
>
>   如果数组中不存在目标值，返回 [-1, -1]。
>
>   示例 1:
>
>   输入: nums = [5,7,7,8,8,10], target = 8
>   输出: [3,4]
>   示例 2:
>
>   输入: nums = [5,7,7,8,8,10], target = 6
>   输出: [-1,-1]

```java
public int[] searchRange(int[] nums, int target) {
        int[] targetRange = {-1, -1};

        int leftIdx = extremeInsertionIndex(nums, target, true);

        // assert that `leftIdx` is within the array bounds and that `target`
        // is actually in `nums`.
        if (leftIdx == nums.length || nums[leftIdx] != target) {
            return targetRange;
        }

        targetRange[0] = leftIdx;
        targetRange[1] = extremeInsertionIndex(nums, target, false)-1;

        return targetRange;
    }
    private int extremeInsertionIndex(int[] nums, int target, boolean left) {
        int lo = 0;
        int hi = nums.length;

        while (lo < hi) {
            int mid = (lo + hi) / 2;
            if (nums[mid] > target || (left && target == nums[mid])) {
                hi = mid;
            }
            else {
                lo = mid+1;
            }
        }

        return lo;
    }
```

# [329. 矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)

>   给定一个整数矩阵，找出最长递增路径的长度。
>
>   对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。
>
>   示例 1:
>
>   输入: nums = 
>   [
>     [9,9,4],
>     [6,6,8],
>     [2,1,1]
>   ] 
>   输出: 4 
>   解释: 最长递增路径为 [1, 2, 6, 9]。
>   示例 2:
>
>   输入: nums = 
>   [
>     [3,4,5],
>     [3,2,6],
>     [2,2,1]
>   ] 
>   输出: 4 
>   解释: 最长递增路径是 [3, 4, 5, 6]。注意不允许在对角线方向上移动。
>

```java
 public int longestIncreasingPath(int[][] matrix) {
        if(matrix == null||matrix.length==0||matrix[0].length == 0)return 0;
        int len = matrix.length;
        int wide = matrix[0].length;
        int[][] cache = new int[len][wide];
        int max = 0;
        for(int i = 0;i < len;i++){
            for(int j = 0;j < wide ;j++){
                int length = dfs(matrix,i,j,cache,-1);
                max = Math.max(length,max);
            }
        }
        return max;

    }
    public int dfs(int[][] matrix,int i,int j,int[][]cache,int pre){
        if(i < 0 || j < 0 || i >= matrix.length||j >= matrix[0].length || matrix[i][j] <= pre){
            return 0;
        }
        if(cache[i][j] > 0){
            return cache[i][j];
        }
        int temp = 0;
        int cur = matrix[i][j];
        temp = Math.max(temp,dfs(matrix,i -1,j,cache,cur));
        temp = Math.max(temp,dfs(matrix,i,j-1,cache,cur));
        temp = Math.max(temp,dfs(matrix,i,j+1,cache,cur));
        temp = Math.max(temp,dfs(matrix,i+1,j,cache,cur));
        temp++;
        //
        cache[i][j] += temp;
        return temp;
    }
```

# [739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

>   请根据每日 气温 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要等待的天数。如果气温在这之后都不会升高，请在该位置用 0 来代替。
>
>   例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。
>
>   提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。

可以维护一个存储下标的单调栈，从栈底到栈顶的下标对应的温度列表中的温度依次递减。如果一个下标在单调栈里，则表示尚未找到下一次温度更高的下标。

```java
  public int[] dailyTemperatures(int[] T) {
        int length = T.length;
        int[] ans = new int[length];
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < length; i++) {
            int temperature = T[i];
            while (!stack.isEmpty() && temperature > T[stack.peek()]) {
                int prevIndex = stack.pop();
                ans[prevIndex] = i - prevIndex;
            }
            stack.push(i);
        }
        return ans;
    }
```

# [178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

>   编写一个 SQL 查询来实现分数排名。
>
>   如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。
>
>   +----+-------+
>   | Id | Score |
>   +----+-------+
>   | 1  | 3.50  |
>   | 2  | 3.65  |
>   | 3  | 4.00  |
>   | 4  | 3.85  |
>   | 5  | 4.00  |
>   | 6  | 3.65  |
>   +----+-------+
>   例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：
>
>   +-------+------+
>   | Score | Rank |
>   +-------+------+
>   | 4.00  | 1    |
>   | 4.00  | 1    |
>   | 3.85  | 2    |
>   | 3.65  | 3    |
>   | 3.65  | 3    |
>   | 3.50  | 4    |
>   +-------+------+

```java
select a.Score as Score,
(select count(distinct b.Score) from Scores as b where b.Score>=a.score)as Rank
from Scores as a
order by a.Score desc;
```

#[169. 多数元素](https://leetcode-cn.com/problems/majority-element/)

>   给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。
>
>   你可以假设数组是非空的，并且给定的数组总是存在多数元素。
>
>    
>
>   示例 1:
>
>   输入: [3,2,3]
>   输出: 3
>   示例 2:
>
>   输入: [2,2,1,1,1,2,2]
>   输出: 2
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/majority-element
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
//Boyer-Moore 投票算法
 public int majorityElement(int[] nums) {
        int count = 0;
        Integer candidate = null;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;
            }
            count += (num == candidate) ? 1 : -1;
        }

        return candidate;
    }
```

# [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

>   给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。
>
>   给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/original_images/17_telephone_keypad.png)
>
>   示例:
>
>   输入："23"
>   输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
 public List<String> letterCombinations(String digits) {
         if(digits.length() == 0) return new ArrayList<String>();
		  char[][] numChar = new char[][]{
			  {'a','b','c'},  
			  {'d','e','f'},    
			  {'g','h','i'},    
			  {'j','k','l'},    
			  {'m','n','o'},    
			  {'p','q','r','s'},
			  {'t','u','v'},    
			  {'w','x','y','z'}			  
		  };
		  List<String> result = new ArrayList<>();
		  char[] tem  = new char[digits.length()];
		  char[][] temp = new char[digits.length()][];
		  for(int i = 0; i < digits.length(); i ++){
			  temp[i] = numChar[digits.charAt(i) - '2'];
		  }
		  build(result, temp, tem, 0);
		  return result;
		  
	  }
	  private static void build(List<String> result, char[][] temp, char[] tem, int idx){
		  for(char tt : temp[idx]){
			  tem[idx] = tt;
			  if(idx == temp.length - 1){
				  result.add(new String(tem));
			  }else{
				  build(result, temp, tem, idx + 1);
			  }
		  }
    }
```

# [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

>   给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
>
>   说明：
>
>   你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？
>
>   示例 1:
>
>   输入: [2,2,1]
>   输出: 1
>   示例 2:
>
>   输入: [4,1,2,1,2]
>   输出: 4
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/single-number
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
  public int singleNumber(int[] nums) {
        int res = 0;
        for (int num : nums) res ^= num;
        return res;
    }
```

# [1114. 按序打印](https://leetcode-cn.com/problems/print-in-order/)

>   我们提供了一个类：
>
>   public class Foo {
>     public void first() { print("first"); }
>     public void second() { print("second"); }
>     public void third() { print("third"); }
>   }
>   三个不同的线程将会共用一个 Foo 实例。
>
>   线程 A 将会调用 first() 方法
>   线程 B 将会调用 second() 方法
>   线程 C 将会调用 third() 方法
>   请设计修改程序，以确保 second() 方法在 first() 方法之后被执行，third() 方法在 second() 方法之后被执行。
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/print-in-order
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
class Foo {
    private CountDownLatch second = new CountDownLatch(1);
    private CountDownLatch third = new CountDownLatch(1);
    public Foo() {

    }

    public void first(Runnable printFirst) throws InterruptedException {

        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        second.countDown();
    }

    public void second(Runnable printSecond) throws InterruptedException {

        second.await();
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        third.countDown();
    }

    public void third(Runnable printThird) throws InterruptedException {

        third.await();
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
```

# [82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

>   给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 没有重复出现 的数字。
>
>   示例 1:
>
>   输入: 1->2->3->3->4->4->5
>   输出: 1->2->5
>   示例 2:
>
>   输入: 1->1->1->2->3
>   输出: 2->3
>
>   来源：力扣（LeetCode）
>   链接：https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii
>   著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

```java
 public ListNode deleteDuplicates(ListNode head) {
   			//baseCase
        if (head == null || head.next == null) {
            return head;
        }

        ListNode next = head.next;
        //如果是这种情况
        //      1 --> 1 --> 1 --> 2 --> 3
        //     head  next
        //1.则需要移动next直到出现与当前head.value不相等的情况（含null）
        //2.并且此时的head已经不能要了，因为已经head是重复的节点
        //--------------else-------------
        //      1 --> 2 --> 3
        //     head  next
        //3.如果没有出现1的情况，则递归返回的节点就作为head的子节点
        if (head.val == next.val) {
            //1
            while (next != null && head.val == next.val) {
                next = next.next;
            }
            //2
            head = deleteDuplicates(next);
        } else {
            //3
            head.next = deleteDuplicates(next);
        }
        return head;
    }
```

