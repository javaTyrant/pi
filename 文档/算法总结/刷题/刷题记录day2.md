[toc]

全是经典题哈哈哈

# [7. 整数反转](https://leetcode-cn.com/problems/reverse-integer/)

>  
>  给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
>
>  **示例 1:**
>
>  ```
>  输入: 123
>  输出: 321
>  ```
>
>   **示例 2:**
>
>  ```
>  输入: -123
>  输出: -321
>  ```
>
>  **示例 3:**
>
>  ```java
>  输入: 120
>  输出: 21
>  ```

核心实现

  int pop = x % 10;

  x /= 10;

 rev = rev * 10 + pop;

```java
  //如何防止溢出
	public int reverse(int x) {
        int rev = 0;
        while (x != 0) {
          	//
            int pop = x % 10;
          	//
            x /= 10;
          	//正数溢出
            if (rev > Integer.MAX_VALUE/10 
                || (rev == Integer.MAX_VALUE / 10 && pop > 7))
              return 0;
          	//负数溢出
            if (rev < Integer.MIN_VALUE/10 
                || (rev == Integer.MIN_VALUE / 10 && pop < -8)) 
              return 0;
          	//
            rev = rev * 10 + pop;
        }
        return rev;
    }
```

# [31. 下一个排列](https://leetcode-cn.com/problems/next-permutation/)

>  实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。
>
>  如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。
>
>  必须原地修改，只允许使用额外常数空间。
>
>  以下是一些例子，输入位于左侧列，其相应输出位于右侧列。
>  1,2,3 → 1,3,2
>  3,2,1 → 1,2,3
>  1,1,5 → 1,5,1

```java
 public void nextPermutation(int[] nums) {
     if(nums.length <= 1){
            return;
        }
        int i = nums.length - 2;
        // 找到第一个下降点，我们要把这个下降点的值增加一点点
        // 对于511这种情况，要把前面两个1都跳过，所以要包含等于
        while(i >= 0 && nums[i] >= nums[i + 1]){
            i--;
        }
        // 如果这个下降点还在数组内，我们找到一个比它稍微大一点的数替换
        // 如果在之外，说明整个数组是降序的，是全局最大了
        if(i >= 0){
            int j = nums.length - 1;
            // 对于151，这种情况，要把最后面那个1跳过，所以要包含等于
            while(j > i && nums[j] <= nums[i]){
                j--;
            }
            swap(nums, i, j);
        }
        // 将下降点之前的部分倒序构成一个最小序列
        reverse(nums, i + 1, nums.length - 1);
    }
       private void swap(int[] nums, int i, int j){
        int tmp = nums[j];
        nums[j] = nums[i];
        nums[i] = tmp;
    }
    
    private void reverse(int[] nums, int left, int right){
        while(left < right){
            swap(nums, left, right);
            left++;
            right--;
        }
    }
```

# [199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/) bfs,递归已搞定

>  
>  给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。
>
>  **示例:**
>
>  ```java
>  输入: [1,2,3,null,5,null,4]
>  输出: [1, 3, 4]
>  解释:
>  
>     1            <---
>   /   \
>  2     3         <---
>   \     \
>    5     4       <---
>  ```
>

#### 递归,也是dfs

```java
 public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        findRightView(root,res,0);
        return res;
    }
    private void findRightView(TreeNode root,List<Integer> res,int depth){
      	//base case
        if(root == null){
            return;
        }
      	//need update res核心步骤
      	//depth和res的关系何在
        if(depth == res.size()){
            res.add(root.val);
        }
      	//深度++
        depth++;
      	//note先找右边
      	//find right
        findRightView(root.right,res,depth);
      	//find left
        findRightView(root.left,res,depth);
    }
```

#### 深度优先搜索

- 时间复杂度 : $${O}(n)$$。

- 空间复杂度 :$$ O(n)$$。

```java
 public List<Integer> rightSideView(TreeNode root) {
        Map<Integer, Integer> rightmostValueAtDepth = new HashMap<Integer, Integer>();
        int max_depth = -1;
				//
        Stack<TreeNode> nodeStack = new Stack<TreeNode>();
        Stack<Integer> depthStack = new Stack<Integer>();
        nodeStack.push(root);
        depthStack.push(0);

        while (!nodeStack.isEmpty()) {
            TreeNode node = nodeStack.pop();
            int depth = depthStack.pop();

            if (node != null) {
            	// 维护二叉树的最大深度
                max_depth = Math.max(max_depth, depth);

                // 如果不存在对应深度的节点我们才插入
                if (!rightmostValueAtDepth.containsKey(depth)) {
                    rightmostValueAtDepth.put(depth, node.val);
                }

                nodeStack.push(node.left);
                nodeStack.push(node.right);
                depthStack.push(depth+1);
                depthStack.push(depth+1);
            }
        }

        List<Integer> rightView = new ArrayList<Integer>();
        for (int depth = 0; depth <= max_depth; depth++) {
            rightView.add(rightmostValueAtDepth.get(depth));
        }

        return rightView;
    }
```

#### 广度优先搜索

两种区别是先加右边还是先加左边

- 时间复杂度 : $${O}(n)$$。

- 空间复杂度 :$$ O(n)$$。

```java
public List<Integer> rightSideView(TreeNode root) {
         List<Integer> res = new ArrayList<>();
        if (root == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
                if (i == size - 1) {  //将当前层的最后一个节点放入结果列表
                    res.add(node.val);
                }
            }
        }
        return res;
    }
```

```java
 public List<Integer> rightSideView(TreeNode root) {
        //res
        List<Integer> result = new ArrayList();
   			if (root == null) return result;
   			//queue
        Queue<TreeNode> queue = new LinkedList();
   			//先offer
        queue.offer(root);
   			//基操判断
        while (!queue.isEmpty()) {
          	//queue的大小一直在边,所以要先取出来
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TreeNode cur = queue.poll();
                // 第一个元素是这一层最右边的元素
                if (i == 0) result.add(cur.val);
                // 相当于从右往左加入
                if (cur.right != null) queue.offer(cur.right);
                if (cur.left != null) queue.offer(cur.left);
            }
        }
        return result;
    }
```



# [215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/) 有趣,已征服

>  在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。
>
>  示例 1:
>
>  输入: [3,2,1,5,6,4] 和 k = 2
>  输出: 5
>  示例 2:
>
>  输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
>  输出: 4
>  说明:
>
>  你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。
>
>  ```sql
>  3, 1, 2, 5, 7, 4, 6
>  是如何执行的
>  quickSort(nums,0,7-1,2)
>  开始quickSort
>  l = 0,r = 6,k = 2
>  l >= r吗 否
>  int i = 0 - 1 = -1, r = 6 + 1 = 7
>  
>  ```
>
>  

```java
public int findKthLargest(int[] nums, int k) {
        return quickSort(nums, 0, nums.length - 1, k);
    }
     static int quickSort(int[] nums, int l, int r, int k) {
        if (l >= r) return nums[l];
        int i = l - 1, j = r + 1;
        j = partition(nums, nums[(l + r) / 2], i, j);
        if (r - j >= k) 
          	return quickSort(nums, j + 1, r, k);
        return quickSort(nums, l, j, k - (r - j));
    }

    private static int partition(int[] nums, int mid, int i, int j) {
        while (i < j) {
            do {
                i++;
            } while (nums[i] < mid);
            do {
                j--;
            } while (nums[j] > mid);
          	//
            if (i < j) swap(nums, i, j);
        }
        return j;
    }

    static void swap(int[] nums, int l, int r) {
        int tmp = nums[l];
        nums[l] = nums[r];
        nums[r] = tmp;
    }
```

# [93. 复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/) 经典

>  给定一个只包含数字的字符串，复原它并返回所有可能的 IP 地址格式。
>
>  有效的 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。
>
>  例如："0.1.2.201" 和 "192.168.1.1" 是 有效的 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效的 IP 地址。
>
>   
>
>  示例 1：
>
>  输入：s = "25525511135"
>  输出：["255.255.11.135","255.255.111.35"]
>  示例 2：
>
>  输入：s = "0000"
>  输出：["0.0.0.0"]
>  示例 3：
>
>  输入：s = "1111"
>  输出：["1.1.1.1"]
>  示例 4：
>
>  输入：s = "010010"
>  输出：["0.10.0.10","0.100.1.0"]
>  示例 5：
>
>  输入：s = "101023"
>  输出：["1.0.10.23","1.0.102.3","10.1.0.23","10.10.2.3","101.0.2.3"]

```java
    public List<String> restoreIpAddresses(String s) {
        List<String> ans = new ArrayList<>();
        back(s,ans,new ArrayList<>(),0);
        return ans;
    }
    private void back(String s,List<String> ans, List<String> cur,int pos){
        if(cur.size()>=4){
            if(s.length() == pos){
                ans.add(String.join(".",cur));
            }
            return;
        }
        for(int i = 1;i<=3;i++){
            if(pos + i > s.length()) break;
            String seg = s.substring(pos,pos+i);
            if(seg.startsWith("0") && seg.length() > 1 || (i == 3 && Integer.parseInt(seg)>255))
                continue;
            cur.add(seg);
            //注意是+i 不是加1
            back(s,ans,cur,pos+i);
            cur.remove(cur.size()-1);    
        }
    }
```

# [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/) 经典

>  给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
>
>  你可以对一个单词进行如下三种操作：
>
>  插入一个字符
>  删除一个字符
>  替换一个字符
>
>
>  示例 1：
>
>  输入：word1 = "horse", word2 = "ros"
>  输出：3
>  解释：
>  horse -> rorse (将 'h' 替换为 'r')
>  rorse -> rose (删除 'r')
>  rose -> ros (删除 'e')
>  示例 2：
>
>  输入：word1 = "intention", word2 = "execution"
>  输出：5
>  解释：
>  intention -> inention (删除 't')
>  inention -> enention (将 'i' 替换为 'e')
>  enention -> exention (将 'n' 替换为 'x')
>  exention -> exection (将 'n' 替换为 'c')
>  exection -> execution (插入 'u')

```java
			//想明白其实很简单
    public int minDistance(String word1, String word2) {
        int n = word1.length();
        int m = word2.length();
        // 有一个字符串为空串
        if (n * m == 0) {
            return n + m;
        }
        //dp数组,搞清楚dp[i][j]代表的含义
        int[][] dp = new int[n + 1][m + 1];
        // 边界状态初始化
        for (int i = 0; i < n + 1; i++) {
            dp[i][0] = i;
        }
        for (int j = 0; j < m + 1; j++) {
            dp[0][j] = j;
        }
        // 计算所有 DP 值
        for (int i = 1; i < n + 1; i++) {
            for (int j = 1; j < m + 1; j++) {
                int left = dp[i - 1][j] + 1;
                int down = dp[i][j - 1] + 1;
                int leftDown = dp[i - 1][j - 1];
                if (word1.charAt(i - 1) != word2.charAt(j - 1)) {
                    leftDown += 1;
                }
                dp[i][j] = Math.min(left, Math.min(down, leftDown));
            }
        }
        return dp[n][m];
    }
```

# [124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

>  给定一个非空二叉树，返回其最大路径和。
>
>  本题中，路径被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。该路径至少包含一个节点，且不一定经过根节点。
>
>   
>
>  示例 1：
>
>  输入：[1,2,3]
>
>         1
>        / \
>       2   3
>
>  输出：6
>  示例 2：
>
>  输入：[-10,9,20,null,null,15,7]
>
>     -10
>     / \
>    9  20
>      /  \
>     15   7
>
>  输出：42
>

```java
 		private int ret = Integer.MIN_VALUE;
    
    public int maxPathSum(TreeNode root) {
        /**
        对于任意一个节点, 如果最大和路径包含该节点, 那么只可能是两种情况:
        1. 其左右子树中所构成的和路径值较大的那个加上该节点的值后向父节点回溯构成最大路径
        2. 左右子树都在最大路径中, 加上该节点的值构成了最终的最大路径
        **/
        getMax(root);
        return ret;
    }
    
    private int getMax(TreeNode r) {
        if(r == null) return 0;
        int left = Math.max(0, getMax(r.left)); // 如果子树路径和为负则应当置0表示最大路径不包含子树
        int right = Math.max(0, getMax(r.right));
        ret = Math.max(ret, r.val + left + right); // 判断在该节点包含左右子树的路径和是否大于当前最大路径和
        return Math.max(left, right) + r.val;
    }
```

# [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/) 经典

>  给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
>  岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
>  此外，你可以假设该网格的四条边均被水包围。
>
>   
>
>  示例 1：
>
>  输入：grid = [
>    ["1","1","1","1","0"],
>    ["1","1","0","1","0"],
>    ["1","1","0","0","0"],
>    ["0","0","0","0","0"]
>  ]
>  输出：1
>  示例 2：
>
>  输入：grid = [
>    ["1","1","0","0","0"],
>    ["1","1","0","0","0"],
>    ["0","0","1","0","0"],
>    ["0","0","0","1","1"]
>  ]
>  输出：3

```java
   public static int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) {
            return 0;
        }
        int x = grid.length;
        int y = grid[0].length;
        int count = 0;
        for (int i = 0; i < x; ++i) {
            for (int j = 0; j < y; ++j) {
                if (grid[i][j] == '1') {
                    ++count;
                    dfs(grid, i, j);
                }
            }
        }
        return count;
    }

    private static void dfs(char[][] grid, int i, int j) {
        int x = grid.length;
        int y = grid[0].length;
      	//核心判断
        if (i < 0 || j < 0 || i >= x || j >= y || grid[i][j] == '0') {
            return;
        }
      	//基操置0
        grid[i][j] = '0';
        dfs(grid, i - 1, j);
        dfs(grid, i + 1, j);
        dfs(grid, i, j - 1);
        dfs(grid, i, j + 1);
    }
```

# [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

>  给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。

```java
public ListNode sortList(ListNode head) {
         if (head == null || head.next == null)
            return head;

        // 使用两个指针，一快一慢，当快指针走到链表尾部时，慢指针正好到中间位置。
        ListNode prev = null, slow = head, fast = head;

        while (fast != null && fast.next != null) {
            prev = slow;
            slow = slow.next;
            fast = fast.next.next;
        }
        //prev表示第一个链表的尾部，所以指向空。slow为第二个指针的头部
        prev.next = null;

        // 继续分裂
        ListNode l1 = sortList(head);
        ListNode l2 = sortList(slow);

        // 当分裂到最后每个元素时，开始两两合并
        return merge(l1, l2);
    }
     //将两个有序链表合并的函数
    ListNode merge(ListNode l1, ListNode l2) {
        ListNode l = new ListNode(0), p = l;

        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                p.next = l1;
                l1 = l1.next;
            } else {
                p.next = l2;
                l2 = l2.next;
            }
            p = p.next;
        }

        if (l1 != null)
            p.next = l1;

        if (l2 != null)
            p.next = l2;

        return l.next;
    }
```

# [22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

>  输入：n = 3
>  输出：[
>         "((()))",
>         "(()())",
>         "(())()",
>         "()(())",
>         "()()()"
>       ]

```java
    List<String> res = new ArrayList<>();
    public List<String> generateParenthesis(int n) {
        dfs(n, n, "");
        return res;
    }

    private void dfs(int left, int right, String curStr) {
        if (left == 0 && right == 0) { // 左右括号都不剩余了，递归终止
            res.add(curStr);
            return;
        }

        if (left > 0) { // 如果左括号还剩余的话，可以拼接左括号
            dfs(left - 1, right, curStr + "(");
        }
        if (right > left) { // 如果右括号剩余多于左括号剩余的话，可以拼接右括号
            dfs(left, right - 1, curStr + ")");
        }
    }
```

# [440. 字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/) 难死爹了

>  输入:
>  n: 13   k: 2
>
>  输出:
>  10
>
>  解释:
>  字典序的排列是 [1, 10, 11, 12, 13, 2, 3, 4, 5, 6, 7, 8, 9]，所以第二小的数字是 10。

```java
 public int findKthNumber(int n, int k) {
        int cur = 1;
        k--;
        while (k > 0) {
            long step = 0, first = cur, last = cur + 1;
            while (first <= n) {
                step += Math.min(last, (long) (n + 1)) - first;
                first *= 10;
                last *= 10;
            }

            if (step > k) {
                //在树里
                k--;
                cur *= 10;
            }
            if (step <= k) {
                //不在树里
                k -= step;
                ++cur;
            }
        }
        return cur;
    }
```

# [54. 螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/) 四个指针

>  
>  给定一个包含 *m* x *n* 个元素的矩阵（*m* 行, *n* 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。
>
>  **示例 1:**
>
>  ```
>  输入:
>  [
>   [ 1, 2, 3 ],
>   [ 4, 5, 6 ],
>   [ 7, 8, 9 ]
>  ]
>  输出: [1,2,3,6,9,8,7,4,5]
>  ```
>
>  **示例 2:**
>
>  ```
>  输入:
>  [
>    [1, 2, 3, 4],
>    [5, 6, 7, 8],
>    [9,10,11,12]
>  ]
>  输出: [1,2,3,4,8,12,11,10,9,5,6,7]
>  ```

```java
public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new ArrayList<>();        
        if(matrix == null||matrix.length == 0||matrix[0].length==0){
            return res;
        }
        int u = 0;
        int d = matrix.length - 1;
        int l = 0;
        int r = matrix[0].length-1;
        while(u <= d && l <= r){
          	//先加一层
            for(int i = l;i<= r;i++){
                res.add(matrix[u][i]);
            }
            u++;
          	//
            for(int i = u; i <= d;i++){
                res.add(matrix[i][r]);
            }
            r--;
          	//反方向,两个边界
            for(int i = r;i >= l && u <= d;i--){
                res.add(matrix[d][i]);
            }
            d--;
          	//反方向,两个边界
            for(int i = d;i >=u && l <= r;i--){
                res.add(matrix[i][l]);
            }
            l++;
        }
        return res;
    }
```

# [从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) 更透彻的了解二叉树吧

>  根据一棵树的前序遍历与中序遍历构造二叉树。
>
>  注意:
>  你可以假设树中没有重复的元素。
>
>  例如，给出
>
>  前序遍历 preorder = [3,9,20,15,7]
>  中序遍历 inorder = [9,3,15,20,7]
>  返回如下的二叉树：
>
>      3
>     / \
>    9  20
>      /  \
>     15   7

```java
 private Map<Integer, Integer> indexMap;

    public TreeNode myBuildTree(int[] preorder, int[] inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {
        if (preorder_left > preorder_right) {
            return null;
        }

        // 前序遍历中的第一个节点就是根节点
        int preorder_root = preorder_left;
        // 在中序遍历中定位根节点
        int inorder_root = indexMap.get(preorder[preorder_root]);
        
        // 先把根节点建立出来
        TreeNode root = new TreeNode(preorder[preorder_root]);
        // 得到左子树中的节点数目
        int size_left_subtree = inorder_root - inorder_left;
        // 递归地构造左子树，并连接到根节点
        // 先序遍历中「从 左边界+1 开始的 size_left_subtree」个元素就对应了中序遍历中「从 左边界 开始到 根节点定位-1」的元素
        root.left = myBuildTree(preorder, inorder, preorder_left + 1, preorder_left + size_left_subtree, inorder_left, inorder_root - 1);
        // 递归地构造右子树，并连接到根节点
        // 先序遍历中「从 左边界+1+左子树节点数目 开始到 右边界」的元素就对应了中序遍历中「从 根节点定位+1 到 右边界」的元素
        root.right = myBuildTree(preorder, inorder, preorder_left + size_left_subtree + 1, preorder_right, inorder_root + 1, inorder_right);
        return root;
    }

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int n = preorder.length;
        // 构造哈希映射，帮助我们快速定位根节点
        indexMap = new HashMap<Integer, Integer>();
        for (int i = 0; i < n; i++) {
            indexMap.put(inorder[i], i);
        }
        return myBuildTree(preorder, inorder, 0, n - 1, 0, n - 1);
    }
```

# [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/) 经典升级

>  反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。
>
>  说明:
>  1 ≤ m ≤ n ≤ 链表长度。
>
>  示例:
>
>  输入: 1->2->3->4->5->NULL, m = 2, n = 4
>  输出: 1->4->3->2->5->NULL

```java
   public ListNode reverseBetween(ListNode head, int m, int n) {
         if(head == null || head.next == null) {
            return head;
        }
        if(m >= n) {
            return head;
        }
        ListNode dh = new ListNode(0);
        dh.next = head;
        ListNode mPre = dh;
        ListNode nNext;
        
        ListNode cur;
        ListNode next;
        
        for(int i=0; i<m-1; i++) {
            mPre = mPre.next;
        }
        cur = mPre.next;
        next = cur.next;
        nNext = next;
        for(int i=0;i<n-m;i++) {
            nNext = next.next;
            next.next = cur;
            cur = next;
            next = nNext;
        }
        mPre.next.next = nNext;
        mPre.next = cur;
        return dh.next;
    }
```

# [102. 二叉树的层次遍历 ](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/) 基操

>  给你一个二叉树，请你返回其按 层序遍历 得到的节点值。 （即逐层地，从左到右访问所有节点）。
>
>   
>
>  示例：
>  二叉树：[3,9,20,null,null,15,7],
>
>      3
>     / \
>    9  20
>      /  \
>     15   7
>  返回其层次遍历结果：
>
>  [
>    [3],
>    [9,20],
>    [15,7]
>  ]

```java
  public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return res;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        int level = 0;
        while(!queue.isEmpty()){
            res.add(new ArrayList<>());
            int size = queue.size();
            for(int i = 0;i< size;i++){
                TreeNode cur = queue.remove();
                res.get(level).add(cur.val);
                if(cur.left !=null){
                    queue.offer(cur.left);
                }
                if(cur.right != null){
                    queue.offer(cur.right);
                }
            }
            level++;
        }
        return res;
    }
```

# [32. 最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses/) 有趣

>  给定一个只包含 '(' 和 ')' 的字符串，找出最长的包含有效括号的子串的长度。
>
>  示例 1:
>
>  输入: "(()"
>  输出: 2
>  解释: 最长有效括号子串为 "()"
>  示例 2:
>
>  输入: ")()())"
>  输出: 4
>  解释: 最长有效括号子串为 "()()"

```java
 public int longestValidParentheses(String s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * right);
            } else if (right >= left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = s.length() - 1; i >= 0; i--) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * left);
            } else if (left >= right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
```

# [19. 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/) 基操,双指针的另一种妙用

>  给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。
>
>  示例：
>
>  给定一个链表: 1->2->3->4->5, 和 n = 2.
>
>  当删除了倒数第二个节点后，链表变为 1->2->3->5.
>

```java
  	//我们可以设想假设设定了双指针 p 和 q 的话，当 q 指向末尾的 NULL，
    //p 与 q 之间相隔的元素个数为 n 时，那么删除掉 p 的下一个指针就完成了要求。
    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (n == 0) return head;
        //复制一个
        ListNode dummy = new ListNode(0);
        dummy.next = head;

        ListNode pre = dummy, cur = dummy;
      	//先移动cur
        while( n >= 0 && cur != null) {
            cur = cur.next;
            n --;
        }
        //再一起移动,cur和pre
        while (cur != null) {
            pre = pre.next;
            cur = cur.next;
        }
        //开始删除吧
        pre.next = pre.next.next;
        //
        return dummy.next;    
    }
```

# [24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/) 基操

>  给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。
>
>  **你不能只是单纯的改变节点内部的值**，而是需要实际的进行节点交换。
>
>  ```java
>  输入：head = [1,2,3,4]
>  输出：[2,1,4,3]
>  输入：head = []
>  输出：[]
>  ```

```java
public ListNode swapPairs(ListNode head) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        
        head = dummy;
        while (head.next != null && head.next.next != null) {
            ListNode n1 = head.next, n2 = head.next.next;
            // head->n1->n2->...
            // => head->n2->n1->...
            head.next = n2;
            n1.next = n2.next;
            n2.next = n1;
            
            // move to next pair
            head = n1;
        }
        
        return dummy.next;
    }
```

# [103. 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

>  给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。
>
>  例如：
>  给定二叉树 [3,9,20,null,null,15,7],
>
>  ​    3
>
>     / \
>    9  20
>      /  \
>     15   7
>  返回锯齿形层次遍历如下：
>
>  [
>    [3],
>    [20,9],
>    [15,7]
>  ]

```java
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
          List<List<Integer>> result = new ArrayList<>();
        if(root == null) return result;
        LinkedList<TreeNode> nodelist = new LinkedList<TreeNode>();
        boolean order = true;
        nodelist.offer(root);
        while(!nodelist.isEmpty()){
            int size = nodelist.size();
            ArrayList<Integer> al = new ArrayList<Integer>();
            for(int i = 0; i < size; i++){
                TreeNode node = nodelist.poll();
                if(order) al.add(node.val);
                else al.add(0,node.val);
                if(node.left != null) nodelist.offer(node.left);
                if(node.right != null) nodelist.offer(node.right);
            }
            order = !order;
            result.add(al);
        }
        return result;
    }
```

# [143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)

>  注意到目标链表即为将原链表的左半端和反转后的右半端合并后的结果。
>
>  这样我们的任务即可划分为三步：
>
>  - 找到原链表的中点（参考「876. 链表的中间结点」）。
>    - 我们可以使用快慢指针来 $$O(N)$$ 地找到链表的中间节点。
>  - 将原链表的右半端反转（参考「206. 反转链表」）。
>    - 我们可以使用迭代法实现链表的反转。
>
>  - 将原链表的两端合并。
>    - 因为两链表长度相差不超过 11，因此直接合并即可。

```java
public void reorderList(ListNode head) {
        if (head == null) {
            return;
        }
        ListNode mid = middleNode(head);
        ListNode l1 = head;
        ListNode l2 = mid.next;
        mid.next = null;
        l2 = reverseList(l2);
        mergeList(l1, l2);
    }

    public ListNode middleNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }

    public void mergeList(ListNode l1, ListNode l2) {
        ListNode l1_tmp;
        ListNode l2_tmp;
        while (l1 != null && l2 != null) {
            l1_tmp = l1.next;
            l2_tmp = l2.next;

            l1.next = l2;
            l1 = l1_tmp;

            l2.next = l1;
            l2 = l2_tmp;
        }
    }
```

