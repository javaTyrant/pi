[toc]

#[350. 两个数组的交集 II](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/) 

>   给定两个数组，编写一个函数来计算它们的交集。
>
>   说明：
>
>   输出结果中每个元素出现的次数，应与元素在两个数组中出现次数的最小值一致。
>   我们可以不考虑输出结果的顺序。
>   进阶：
>
>   如果给定的数组已经排好序呢？你将如何优化你的算法？
>   如果 nums1 的大小比 nums2 小很多，哪种方法更优？
>   如果 nums2 的元素存储在磁盘上，内存是有限的，并且你不能一次加载所有的元素到内存中，你该怎么办？

时间复杂度：O(m+n)*O*(*m*+*n*)

空间复杂度：O(min(m,n))*O*(min(*m*,*n*))

```java
 public int[] intersect(int[] nums1, int[] nums2) {
   			//巧妙的换一下位置,确保nums1.length < nums2.length
        if (nums1.length > nums2.length) {
            return intersect(nums2, nums1);
        }
   			//构建一个map
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int num : nums1) {
          	//统计个数
            int count = map.getOrDefault(num, 0) + 1;
            map.put(num, count);
        }
   			//构建交集数组,还不知道大小,先以小数组的长度为准
        int[] intersection = new int[nums1.length];
   			//索引,保存的索引
        int index = 0;
   			//去nums2里面找
        for (int num : nums2) {
          	//找出数量
            int count = map.getOrDefault(num, 0);
          	//如果存在
            if (count > 0) {
              	//保存数组
                intersection[index++] = num;
              	//count--
                count--;
              	//更新下count
                if (count > 0) {
                    map.put(num, count);
                } else {
                  	//否则移除
                    map.remove(num);
                }
            }
        }
   			//截取数组
        return Arrays.copyOfRange(intersection, 0, index);
    }
```

排序

时间复杂度：O(m log m+nlog n)*O*(*m*log*m*+*n*log*n*)

空间复杂度：O(min(m,n))*O*(min(*m*,*n*))

```java
 public int[] intersect(int[] nums1, int[] nums2) {
   			//先排序
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int length1 = nums1.length, length2 = nums2.length;
   			//结果
        int[] intersection = new int[Math.min(length1, length2)];
   			//
        int index1 = 0, index2 = 0, index = 0;
   			//双指针
        while (index1 < length1 && index2 < length2) {
            if (nums1[index1] < nums2[index2]) {
                index1++;
            } else if (nums1[index1] > nums2[index2]) {
                index2++;
            } else {
              	//
                intersection[index] = nums1[index1];
                index1++;
                index2++;
                index++;
            }
        }
        return Arrays.copyOfRange(intersection, 0, index);
    }
```



#[257. 二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/)

>   给定一个二叉树，返回所有从根节点到叶子节点的路径。
>
>   **说明:** 叶子节点是指没有子节点的节点。

```java
 public List<String> binaryTreePaths(TreeNode root) {
        List<String> ret = new ArrayList<>();
        dfs(root, new StringBuilder(), ret);
        return ret;
    }
    private void dfs(TreeNode root, StringBuilder tmp, List<String> ret) {
        if (root == null) {
            return ;
        }
      	//
        String s = tmp.length() == 0 ? "" : "->";
        tmp.append(s);
        tmp.append(root.val);
        if (root.left == null && root.right == null) {
            ret.add(tmp.toString());
            return ;
        }
        dfs(root.left, new StringBuilder(tmp), ret);
        dfs(root.right, new StringBuilder(tmp), ret);
    }
```

#[剑指 Offer 12 - 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

>   请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。
>
>   [["a","b","c","e"],
>   ["s","f","c","s"],
>   ["a","d","e","e"]]
>
>   但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。
>
>   输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
>   输出：true
>
>   **示例 2：**
>
>   ```java
>   输入：board = [["a","b"],["c","d"]], word = "abcd"
>   输出：false
>   ```

```java
   public boolean exist(char[][] board, String word) {
        char c = word.charAt(0);
        for (int i = 0; i < board.length; i++) {
            for (int j = 0, k = board[i].length - 1; j <= k; j++, k--) {
                if (board[i][j] == c) if (fbs(board, i, j, word, 1)) return true;
                if (j != k && board[i][k] == c) if (fbs(board, i, k, word, 1)) return true;
            }
        }
        return false;
    }

    public boolean fbs(char[][] board, int i, int j, String word, int idx) {
        if (idx >= word.length()) return true;

        char ch = board[i][j];
        board[i][j] = '-';
        char c = word.charAt(idx++);

        if (i - 1 >= 0 && board[i - 1][j] == c) if (fbs(board, i - 1, j, word, idx)) return true;
        if (i + 1 < board.length && board[i + 1][j] == c) if (fbs(board, i + 1, j, word, idx)) return true;

        if (j - 1 >= 0 && board[i][j - 1] == c) if (fbs(board, i, j - 1, word, idx)) return true;
        if (j + 1 < board[i].length && board[i][j + 1] == c) if (fbs(board, i, j + 1, word, idx)) return true;

        board[i][j] = ch;
        return false;
    }
```

#[剑指 Offer 21 - 调整数组顺序使奇数位](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

>   输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。
>
>    
>
>   示例：
>
>   输入：nums = [1,2,3,4]
>   输出：[1,3,2,4] 
>   注：[3,1,2,4] 也是正确的答案之一。

```java
  public int[] exchange(int[] nums) {
        if (nums == null || nums.length == 0) {
            return new int[0];
        }
    		//双指针
        int i = 0, j = nums.length - 1;
        while (i < j) {
          	//i是第一个偶数
            while (i < j && nums[i] % 2 != 0) i++;
          	//j找到最后一个奇数
            while (i < j && nums[j] % 2 == 0) j--;
          	//如果i < j
            if (i < j) {
              	//交换
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j] = temp;
            }
        }
        return nums;
    }
```

#[剑指 Offer 57 - II - 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

>   输入一个正整数 target ，输出所有和为 target 的连续正整数序列（至少含有两个数）。
>
>   序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。
>
>    
>
>   示例 1：
>
>   输入：target = 9
>   输出：[[2,3,4],[4,5]]
>   示例 2：
>
>   输入：target = 15
>   输出：[[1,2,3,4,5],[4,5,6],[7,8]]

返回值很搞心态哈哈

```java
 public int[][] findContinuousSequence(int target) {
        ArrayList<int[]> res = new ArrayList<>();
   			//求半
        for(int i = 1;i<=target/2;i++){
            int sum = 0;
            int j = i;
            while(sum < target){
                sum += j;
                j++;
            }
            if(sum == target){
                int[] sol = new int[j-i];
                for(int k = 0;k<j-i;k++){
                    sol[k] = k+i;
                }
                res.add(sol);
            }
        }
        return res.toArray(new int[res.size()][]);
    }
```

#[184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

>   

```sql
select 
t.Name as Department,
e.Name as Employee, 
e.Salary as Salary
from(
    select DepartmentId, 
        d.name as Name ,
        max(Salary) as Salary from Employee  e
    join Department d on d.Id = e.DepartmentId 
    group by DepartmentId) as t
join Employee e
on t.DepartmentId = e.DepartmentId and e.Salary = t.Salary
```

#[994. 腐烂的橘子](https://leetcode-cn.com/problems/rotting-oranges/)

>   
>   在给定的网格中，每个单元格可以有以下三个值之一：
>
>   - 值 `0` 代表空单元格；
>   - 值 `1` 代表新鲜橘子；
>   - 值 `2` 代表腐烂的橘子。
>
>   每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。
>
>   返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`。
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/16/oranges.png)

```java
 public int orangesRotting(int[][] grid) {
 				int r = grid.length, c = grid[0].length, time = 0;
        Queue<int[]> queue = new LinkedList<>();
        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++)
                if (grid[i][j] == 2)
                    queue.add(new int[]{i, j, 0});
        while (!queue.isEmpty()) {
            int[] temp = queue.poll();
            int i = temp[0], j = temp[1];
            time = temp[2];
            if (i > 0 && grid[i - 1][j] == 1) {
                grid[i - 1][j] = 2;
                queue.add(new int[]{i - 1, j, time + 1});
            }
            if (i < r - 1 && grid[i + 1][j] == 1) {
                grid[i + 1][j] = 2;
                queue.add(new int[]{i + 1, j, time + 1});
            }
            if (j > 0 && grid[i][j - 1] == 1) {
                grid[i][j - 1] = 2;
                queue.add(new int[]{i, j - 1, time + 1});
            }
            if (j < c - 1 && grid[i][j + 1] == 1) {
                grid[i][j + 1] = 2;
                queue.add(new int[]{i, j + 1, time + 1});
            }
        }
        for (int i = 0; i < r; i++)
            for (int j = 0; j < c; j++)
                if (grid[i][j] == 1)
                    return -1;
        return time;
    }
```

```java
   	int[][] grid;
   	int r, b;
    public int orangesRotting(int[][] grid) {
        this.grid = grid;
        r = grid[0].length - 1;
        b = grid.length - 1;
        for (int y = 0; y <= b; y++)
            for (int x = 0; x <= r; x++)
                if (grid[y][x] == 2)
                    dfs(x, y, 2);
        int ans = 2;
        for (int y = 0; y <= b; y++) {
            for (int x = 0; x <= r; x++) {
                if (grid[y][x] == 1)
                    return -1;
                ans = Math.max(ans, grid[y][x]);
            }
        }
        return ans - 2;
    }

    void dfs(int x, int y, int count) {
        int c = grid[y][x];
        if (c == 0 || (c != 1 && c < count)) return;
        grid[y][x] = count++;
        if (x != 0) dfs(x - 1, y, count);
        if (x != r) dfs(x + 1, y, count);
        if (y != 0) dfs(x, y - 1, count);
        if (y != b) dfs(x, y + 1, count);
    }
```



#[862. 和至少为 K 的最短子数组](https://leetcode-cn.com/problems/shortest-subarray-with-sum-at-least-k/)(有趣的一题)

>   返回 A 的最短的非空连续子数组的长度，该子数组的和至少为 K 。
>
>   如果没有和至少为 K 的非空子数组，返回 -1 。 
>
>   示例 1：
>
>   输入：A = [1], K = 1
>   输出：1
>   示例 2：
>
>   输入：A = [1,2], K = 4
>   输出：-1
>   示例 3：
>
>   输入：A = [2,-1,2], K = 3
>   输出：3
>
>
>   提示：
>
>   1 <= A.length <= 50000
>   -10 ^ 5 <= A[i] <= 10 ^ 5
>   1 <= K <= 10 ^ 9

```java
public int shortestSubarray(int[] A, int K) {
        //起点
        int start = 0;
        //最小值的缓存
        int minl = -1;
        //
        int maxw = A.length - 1;
        //和
        int sum = 0;
        for (int i = 0; i < A.length; i++) {
            //极端值判断
            if (A[i] >= K) return 1;
            //
            sum += A[i];
            //负数,和小于0,起反作用,sum归0,指针前进1
            if (sum <= 0) {
                sum = 0;
                start = i + 1;
                continue;
            }
            //
            int k = i - 1;
            while (A[k + 1] < 0 && k >= start) {
                A[k] += A[k + 1];
                A[k + 1] = 0;
                k--;
            }
            if (i - start > maxw) {
                sum -= A[start];
                start++;
            }
            if (sum >= K) {
                int l = i - start + 1;
                for (int j = start; j < i; j++) {
                    sum -= A[j];
                    if (sum < K) {
                        l = i - j + 1;
                        start = j + 1;
                        maxw = i - j;
                        break;
                    }
                }
                if (l < minl || minl < 0) minl = l;
            }
        }
        return minl;
    }
```

# [448. 找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/)

>   
>   给定一个范围在 1 ≤ a[i] ≤ *n* ( *n* = 数组大小 ) 的 整型数组，数组中的元素一些出现了两次，另一些只出现一次。
>
>   找到所有在 [1, *n*] 范围之间没有出现在数组中的数字。
>
>   您能在不使用额外空间且时间复杂度为*O(n)*的情况下完成这个任务吗? 你可以假定返回的数组不算在额外空间内。
>
>   **示例:**
>
>   ```
>   输入:
>   [4,3,2,7,8,2,3,1]
>   
>   输出:
>   [5,6]
>   
>   ```
>
>    使用了额外空间了

```java
  List<Integer> result = new ArrayList<>();
        if (nums == null || nums.length == 0) return result;
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 1; i < nums.length + 1; i++) {
            map.put(i, 1);
        }
        for (int num : nums) {
            map.remove(num);
        }
        result.addAll(map.keySet());
        return result;
    }
```


```java
public List<Integer> findDisappearedNumbers(int[] nums) {      
    for (int i = 0; i < nums.length; i++) {
        int newIndex = Math.abs(nums[i]) - 1;
        if (nums[newIndex] > 0) {
            nums[newIndex] *= -1;
        }
    }
    List<Integer> result = new LinkedList<Integer>();
    for (int i = 1; i <= nums.length; i++) {     
        if (nums[i - 1] > 0) {
            result.add(i);
        }
    }
    return result;
}
```

# [67. 二进制求和](https://leetcode-cn.com/problems/add-binary/)

>   给你两个二进制字符串，返回它们的和（用二进制表示）。
>
>   输入为 非空 字符串且只包含数字 1 和 0。
>
>    
>
>   示例 1:
>
>   输入: a = "11", b = "1"
>   输出: "100"
>   示例 2:
>
>   输入: a = "1010", b = "1011"
>   输出: "10101"

```java
public String addBinary(String a, String b) {
   			int indexa = a.length() - 1;
        int indexb = b.length() - 1;
        char[] chars = new char[Math.max(indexa, indexb) + 2];
        int flag = 0;
        for(int i = chars.length -1; i> 0; i--){
            if(indexa >=0 && indexb >= 0){
                chars[i] = (char)(a.charAt(indexa) + b.charAt(indexb) + flag - '0');
                indexa--;
                indexb--;
            }
            else if(indexa < 0){
                chars[i] = (char)(b.charAt(indexb) + flag );
                indexb--;
            }
            else{
                chars[i] = (char)(a.charAt(indexa) + flag );
                indexa--;
            }
            if(chars[i] >= '2'){
                chars[i] = (char)(chars[i] - 2);
                flag = 1;
                if(i == 1){
                    chars[0] = '1';
                    return new String (chars);
                }
            }
            else
                flag = 0;
        }
        return new String(chars, 1, chars.length - 1);
    }
```

# [118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle/)

>   给定一个非负整数 *numRows，*生成杨辉三角的前 *numRows* 行。
>
>   ```java
>   输入: 5
>   输出:
>   [
>        [1],
>       [1,1],
>      [1,2,1],
>     [1,3,3,1],
>    [1,4,6,4,1]
>   ]
>   ```
>
>   

```java
 public List<List<Integer>> generate(int numRows) {
      List<List<Integer>> res = new ArrayList<>();
        if (numRows == 0) {
            return res;
        }
        if (numRows > 1) {
            res.add(Arrays.asList(1));
        } else {
            res.add(Arrays.asList(1));
            return res;
        }
        if (numRows > 2) {
            res.add(Arrays.asList(1, 1));
        } else {
            res.add(Arrays.asList(1, 1));
            return res;
        }
        for (int i = 3; i <= numRows; i++) {
            List<Integer> inner = new ArrayList<>();
            inner.add(1);
            for (int j = 0; j < i - 2; j++) {
                inner.add(res.get(i - 2).get(j) + res.get(i - 2).get(j + 1));
            }
            inner.add(1);
            res.add(inner);
        }
        return res;
    }
```

# [442. 数组中重复的数据](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)

>   给定一个整数数组 a，其中1 ≤ a[i] ≤ n （n为数组长度）, 其中有些元素出现两次而其他元素出现一次。
>
>   找到所有出现两次的元素。
>
>   你可以不用到任何额外空间并在O(n)时间复杂度内解决这个问题吗？
>
>   示例：
>
>   输入:
>   [4,3,2,7,8,2,3,1]
>
>   输出:
>   [2,3]

```java
   public List<Integer> findDuplicates(int[] nums) {
        List<Integer> ret = new ArrayList<>();
        int n = nums.length;
        for(int i = 0; i < n; i++){
            nums[(nums[i] - 1) % n] += n;
        }
        for(int i = 0; i < n; i++){
            if(nums[i] > 2 * n) ret.add(i+1);
        }
        return ret;
    }
```

# [剑指 Offer 06 - 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

>   输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。
>
>    
>
>   **示例 1：**
>
>   ```java
>   输入：head = [1,3,2]
>   输出：[2,3,1]
>   ```

```java
public int[] reversePrint(ListNode head) {
        Stack<ListNode> stack = new Stack<>();
        ListNode cur = head;
        while(cur != null){
            stack.push(cur);
            cur = cur.next;
        }
        int size = stack.size();
        int[] res = new int[size];
        for(int i = 0;i<size;i++){
            res[i] = stack.pop().val;
        }
        return res;
    }
```

# [剑指 Offer 30 - 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

>   
>   定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。
>
>    
>
>   **示例:**
>
>   ```
>   MinStack minStack = new MinStack();
>   minStack.push(-2);
>   minStack.push(0);
>   minStack.push(-3);
>   minStack.min();   --> 返回 -3.
>   minStack.pop();
>   minStack.top();      --> 返回 0.
>   minStack.min();   --> 返回 -2.
>   ```

```java
class MinStack {

   class Node{
        private int min;
        private int val;
        private Node next;
        public Node(int val,int min,Node node){
            this.min = min;
            this.next = node;
            this.val = val;
        }
        public Node(int min,int val){
            this.min = min;
            this.val = val;
        }
    }
     private Node node;

    /** initialize your data structure here. */
    public MinStack() {

    }
    
    public void push(int x) {
        if(node == null){
            node = new Node(x,x);
        }else{
            node = new Node(x,Math.min(x,node.min),node);
        }
    }
    
    public void pop() {
        node = node.next;
    }
    
    public int top() { 
       return node.val;
    }
    
    public int min() {
       return node.min;
    }
}
```

# [面试题 08.11 - 硬币](https://leetcode-cn.com/problems/coin-lcci/)

>   硬币。给定数量不限的硬币，币值为25分、10分、5分和1分，编写代码计算n分有几种表示法。(结果可能会很大，你需要将结果模上1000000007)
>
>   示例1:
>
>    输入: n = 5
>    输出：2
>    解释: 有两种方式可以凑成总金额:
>   5=5
>   5=1+1+1+1+1
>   示例2:
>
>    输入: n = 10
>    输出：4
>    解释: 有四种方式可以凑成总金额:
>   10=10
>   10=5+5
>   10=5+1+1+1+1+1
>   10=1+1+1+1+1+1+1+1+1+1
>   说明：
>
>   注意:
>
>   你可以假设：
>
>   0 <= n (总金额) <= 1000000
>

```java
 public int waysToChange(int n) {
            final int MOD = 1000000007;
        int[] coins = new int[] {1, 5, 10, 25};
        int[] dp = new int[n + 1];
        dp[0] = 1;
        for(int i = 0 ; i < coins.length; i++) {
            for(int j = coins[i]; j <= n; j++) {
                dp[j] += dp[j - coins[i]];
                dp[j] %= MOD;
            }
        }
        return dp[n];
    }
```

#[235. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

>   给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。
>
>   百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”
>
>   例如，给定如下二叉搜索树:  root = [6,2,8,0,4,7,9,null,null,3,5]
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/binarysearchtree_improved.png)
>
>    
>
>   示例 1:
>
>   输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
>   输出: 6 
>   解释: 节点 2 和节点 8 的最近公共祖先是 6。
>   示例 2:
>
>   输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
>   输出: 2
>   解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。
>
>
>   说明:
>
>   所有节点的值都是唯一的。
>   p、q 为不同节点且均存在于给定的二叉搜索树中。

```java
  public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    		//base case
        if (p.val==root.val){
            return p;
        } 
        if (q.val==root.val){
            return q;
        }
    		//因为是二叉搜索树,所有有额外的可以使用的信息
    		//如果p.val > root 且 q.val > root
        if (p.val > root.val && q.val > root.val) {
          	//在右边的数
            return lowestCommonAncestor(root.right,p,q);
        }else if (p.val < root.val && q.val < root.val) {
          	否则在左边
            return lowestCommonAncestor(root.left,p,q);
        }else{
          	//否则root就是
            return root;
        }
    }
```

# [403. 青蛙过河](https://leetcode-cn.com/problems/frog-jump/)

>   一只青蛙想要过河。 假定河流被等分为 x 个单元格，并且在每一个单元格内都有可能放有一石子（也有可能没有）。 青蛙可以跳上石头，但是不可以跳入水中。
>
>   给定石子的位置列表（用单元格序号升序表示）， 请判定青蛙能否成功过河（即能否在最后一步跳至最后一个石子上）。 开始时， 青蛙默认已站在第一个石子上，并可以假定它第一步只能跳跃一个单位（即只能从单元格1跳至单元格2）。
>
>   如果青蛙上一步跳跃了 k 个单位，那么它接下来的跳跃距离只能选择为 k - 1、k 或 k + 1个单位。 另请注意，青蛙只能向前方（终点的方向）跳跃。
>
>   请注意：
>
>   石子的数量 ≥ 2 且 < 1100；
>   每一个石子的位置序号都是一个非负整数，且其 < 231；
>   第一个石子的位置永远是0。
>   示例 1:
>
>   [0,1,3,5,6,8,12,17]
>
>   总共有8个石子。
>   第一个石子处于序号为0的单元格的位置, 第二个石子处于序号为1的单元格的位置,
>   第三个石子在序号为3的单元格的位置， 以此定义整个数组...
>   最后一个石子处于序号为17的单元格的位置。
>
>   返回 true。即青蛙可以成功过河，按照如下方案跳跃： 
>   跳1个单位到第2块石子, 然后跳2个单位到第3块石子, 接着 
>   跳2个单位到第4块石子, 然后跳3个单位到第6块石子, 
>   跳4个单位到第7块石子, 最后，跳5个单位到第8个石子（即最后一块石子）。
>   示例 2:
>
>   [0,1,2,3,4,8,9,11]
>
>   返回 false。青蛙没有办法过河。 
>   这是因为第5和第6个石子之间的间距太大，没有可选的方案供青蛙跳跃过去。

看下官方题解优化的思路,很棒

[官方题解](https://leetcode-cn.com/problems/frog-jump/solution/qing-wa-guo-he-by-leetcode/)

```java
   public boolean canCross(int[] stones) {
        HashMap<Integer, Set<Integer>> map = new HashMap<>();
        for (int i = 0; i < stones.length; i++) {
            map.put(stones[i], new HashSet<Integer>());
        }
        map.get(0).add(0);
        for (int i = 0; i < stones.length; i++) {
            for (int k : map.get(stones[i])) {
                for (int step = k - 1; step <= k + 1; step++) {
                    if (step > 0 && map.containsKey(stones[i] + step)) {
                        map.get(stones[i] + step).add(step);
                    }
                }
            }
        }
        return map.get(stones[stones.length - 1]).size() > 0;
    }
```

#[556. 下一个更大元素 III](https://leetcode-cn.com/problems/next-greater-element-iii/)

>   给定一个32位正整数 n，你需要找到最小的32位整数，其与 n 中存在的位数完全相同，并且其值大于n。如果不存在这样的32位整数，则返回-1。
>
>   示例 1:
>
>   输入: 12
>   输出: 21
>   示例 2:
>
>   输入: 21
>   输出: -1

```java
public int nextGreaterElement(int n) {
        String MAX = String.valueOf(Integer.MAX_VALUE);
        // System.out.println(MAX);
        char[] arr = String.valueOf(n).toCharArray();
        int maxv = 0, i = arr.length-1;
        while(i>=0 && maxv<=arr[i]){
            maxv = Math.max(maxv,arr[i]);
            i--;
        }
        if(i<0) return -1;
        //arr[i]是第一个后面有比arr[i]更大的数
        int j = arr.length-1;
        while(arr[j]<=arr[i]) j--;
        //arr[j]是比arr[i]大的数里最靠后的数
        char tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
        //交换arr[i]和arr[j]
        char[] ca = Arrays.copyOfRange(arr,i+1,arr.length);  
        //将arr[i+1:]进行排序
        Arrays.sort(ca);
        //将排序的结果与前面的数组相连并转换成整数
        String res = String.valueOf(Arrays.copyOfRange(arr,0,i+1))+String.valueOf(ca);
        //结果可能溢出，此时应该返回-1
        if(res.length()==MAX.length()){
            for(int k=0;k<res.length();k++){
                if(MAX.charAt(k)<res.charAt(k)) return -1;
            }
        }
        return Integer.valueOf(res);
    }
```

# [704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

>   给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。
>
>
>   示例 1:
>
>   输入: nums = [-1,0,3,5,9,12], target = 9
>   输出: 4
>   解释: 9 出现在 nums 中并且下标为 4
>   示例 2:
>
>   输入: nums = [-1,0,3,5,9,12], target = 2
>   输出: -1
>   解释: 2 不存在 nums 中因此返回 -1

**如何避免死循环**

```java
   public int search(int[] nums, int target) {
        int i = 0,j = nums.length - 1;
     		//为什么是小于等于
        while(i <= j){
            int mid = i + (j - i - 1) / 2;
            if(nums[mid] > target){
              	//为什么要减1
                j = mid - 1;
            }else if(nums[mid] < target){
              	//为什么要加1
                i = mid + 1;
            }else{
                return mid;
            }
        }
        return -1;
    }
```

# [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)

>   给出一个完全二叉树，求出该树的节点个数。
>
>   说明：
>
>   完全二叉树的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层，则该层包含 1~ 2h 个节点。
>
>   示例:
>
>   输入: 
>       1
>      / \
>     2   3
>    / \  /
>   4  5 6
>
>   输出: 6
>

```java
 public int countNodes(TreeNode root) {
        if(root == null){
            return 0;
        }
        return countNodes(root.left) + countNodes(root.right) + 1;
    }
```

