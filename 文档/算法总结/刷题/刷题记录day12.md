[toc]

#[456. 132模式](https://leetcode-cn.com/problems/132-pattern/)

>   给定一个整数序列：a1, a2, ..., an，一个132模式的子序列 ai, aj, ak 被定义为：当 i < j < k 时，ai < ak < aj。设计一个算法，当给定有 n 个数字的序列时，验证这个序列中是否含有132模式的子序列。
>
>   注意：n 的值小于15000。
>
>   示例1:
>
>   输入: [1, 2, 3, 4]
>
>   输出: False
>
>   解释: 序列中不存在132模式的子序列。
>   示例 2:
>
>   输入: [3, 1, 4, 2]
>
>   输出: True
>
>   解释: 序列中有 1 个132模式的子序列： [1, 4, 2].
>

```java
   public boolean find132pattern(int[] nums) {
        int n = nums.length;
        int third = Integer.MIN_VALUE;
        int top = n;
        
        for (int i = n - 1; i >= 0; i--) {
            // Assumes that third is already smaller than some element nums[j] where j is between i and third's index
            // We find a "132" pattern if nums[i] < third
            if (nums[i] < third) return true;
            
            // Now we have nums[i] >= third
            
            // We now view nums[i] as the second element, and increase third as much as possible, but keep third < nums[i].
            // We do this because we want to maximize the chance of finding a "132" pattern in a future iteration.
            // Note that {nums[top], ..., nums[n-1]} is a stack has the following property:
            // nums[top] <= nums[top+1] <= ... <= nums[n-1]
            while (top < n && nums[i] > nums[top]) {
                third = nums[top++];
            }
            
            // Now we have nums[i] <= nums[top] (which indicates that the stack is monotonical)
            // We push nums[i] to the "stack"            
            top--;
            nums[top] = nums[i];
        }
        
        return false;
    }
```

#[295. 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

>   中位数是有序列表中间的数。如果列表长度是偶数，中位数则是中间两个数的平均值。
>
>   例如，
>
>   [2,3,4] 的中位数是 3
>
>   [2,3] 的中位数是 (2 + 3) / 2 = 2.5
>
>   设计一个支持以下两种操作的数据结构：
>
>   void addNum(int num) - 从数据流中添加一个整数到数据结构中。
>   double findMedian() - 返回目前所有元素的中位数。
>   示例：
>
>   addNum(1)
>   addNum(2)
>   findMedian() -> 1.5
>   addNum(3) 
>   findMedian() -> 2
>   进阶:
>
>   如果数据流中所有整数都在 0 到 100 范围内，你将如何优化你的算法？
>   如果数据流中 99% 的整数都在 0 到 100 范围内，你将如何优化你的算法？

```java
class MedianFinder {
    
    PriorityQueue<Integer> minPQ = null;
    PriorityQueue<Integer> maxPQ = null; 

    /** initialize your data structure here. */
    public MedianFinder() {
        minPQ = new PriorityQueue<Integer>();
        maxPQ = new PriorityQueue<Integer>(1,(i1,i2)->{return i2-i1;});
    }
    
    public void addNum(int num) {
        if ( maxPQ.isEmpty() && minPQ.isEmpty() )
            maxPQ.add(num);
        else if ( !maxPQ.isEmpty() && minPQ.isEmpty() )
        {
            if ( num >= maxPQ.peek() )
                minPQ.add(num);
            else
            {
                minPQ.add(maxPQ.poll());
                maxPQ.add(num);
            }
        }
        else if ( maxPQ.isEmpty() && !minPQ.isEmpty() )
        {
            if ( num <= minPQ.peek() )
                maxPQ.add(num);
            else
            {
                maxPQ.add(minPQ.poll());
                minPQ.add(num);
            }
        }
        else
        {
            if ( num <= minPQ.peek() )
                maxPQ.add(num);
            else
                minPQ.add(num);
            while ( maxPQ.size() >= minPQ.size() + 2 )
                minPQ.add(maxPQ.poll());
            while ( maxPQ.size() + 2 <= minPQ.size() )
                maxPQ.add(minPQ.poll());
        }
        // System.out.println(Arrays.toString(maxPQ.toArray()));
        // System.out.println(Arrays.toString(minPQ.toArray()));
    }
    
    public double findMedian() {
        if ( maxPQ.size() > minPQ.size() )
            return maxPQ.peek();
        else if ( maxPQ.size() < minPQ.size() )
            return minPQ.peek();
        else
            return ( maxPQ.peek() + minPQ.peek() ) * 0.5;
    }
}
```

#[109. 有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

>   给定一个单链表，其中的元素按升序排序，将其转换为高度平衡的二叉搜索树。
>
>   本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。
>
>   示例:
>
>   给定的有序链表： [-10, -3, 0, 5, 9],
>
>   一个可能的答案是：[0, -3, 9, -10, null, 5], 它可以表示下面这个高度平衡二叉搜索树：
>
>         0
>        / \
>      -3   9
>      /   /
>    -10  5

```java
 private TreeNode sortedChild(ListNode head, ListNode tail) {
        if (head == tail)
            return null;
        if (head.next == tail) {
            //如果只剩这一个元素了
            TreeNode root = new TreeNode(head.val);
            return root;
        }
        ListNode mid = head, temp = head;
        //--------------------精华部分Begin-----------------------
        while (temp != tail && temp.next != tail) {
            //一个前进一个位置,一个前进2个位置,最后mid会到中间,注意判断条件
            mid = mid.next;
            temp = temp.next.next;
        }
        //--------------------精华部分End-----------------------
        TreeNode root = new TreeNode(mid.val);
        root.left = sortedChild(head, mid);
        root.right = sortedChild(mid.next, tail);
        return root;
     }

    public TreeNode sortedListToBST(ListNode head) {
        return sortedChild(head, null);
    }
```

#[165. 比较版本号](https://leetcode-cn.com/problems/compare-version-numbers/)

>   给你两个版本号 version1 和 version2 ，请你比较它们。
>
>   版本号由一个或多个修订号组成，各修订号由一个 '.' 连接。每个修订号由 多位数字 组成，可能包含 前导零 。每个版本号至少包含一个字符。修订号从左到右编号，下标从 0 开始，最左边的修订号下标为 0 ，下一个修订号下标为 1 ，以此类推。例如，2.5.33 和 0.1 都是有效的版本号。
>
>   比较版本号时，请按从左到右的顺序依次比较它们的修订号。比较修订号时，只需比较 忽略任何前导零后的整数值 。也就是说，修订号 1 和修订号 001 相等 。如果版本号没有指定某个下标处的修订号，则该修订号视为 0 。例如，版本 1.0 小于版本 1.1 ，因为它们下标为 0 的修订号相同，而下标为 1 的修订号分别为 0 和 1 ，0 < 1 。
>
>   返回规则如下：
>
>   如果 version1 > version2 返回 1，
>   如果 version1 < version2 返回 -1，
>   除此之外返回 0。
>
>
>   示例 1：
>
>   输入：version1 = "1.01", version2 = "1.001"
>   输出：0
>   解释：忽略前导零，"01" 和 "001" 都表示相同的整数 "1"
>   示例 2：
>
>   输入：version1 = "1.0", version2 = "1.0.0"
>   输出：0
>   解释：version1 没有指定下标为 2 的修订号，即视为 "0"
>   示例 3：
>
>   输入：version1 = "0.1", version2 = "1.1"
>   输出：-1
>   解释：version1 中下标为 0 的修订号是 "0"，version2 中下标为 0 的修订号是 "1" 。0 < 1，所以 version1 < version2
>   示例 4：
>
>   输入：version1 = "1.0.1", version2 = "1"
>   输出：1
>   示例 5：
>
>   输入：version1 = "7.5.2.4", version2 = "7.5.3"
>   输出：-1
>
>
>   提示：
>
>   1 <= version1.length, version2.length <= 500
>   version1 和 version2 仅包含数字和 '.'
>   version1 和 version2 都是 有效版本号
>   version1 和 version2 的所有修订号都可以存储在 32 位整数 中

```java
public int compareVersion(String version1, String version2) {
        String[] a1 = version1.split("\\.");
        String[] a2 = version2.split("\\.");
        int max = Math.max(a1.length,a2.length);
        for(int i = 0;i < max; i++){
            int one = i < a1.length ? Integer.valueOf(a1[i]) : 0;
            int two = i < a2.length ? Integer.valueOf(a2[i]) : 0;
            if(one < two) return -1;
            if(one > two) return 1;
        }
        return 0;
    }
```

#[66. 加一](https://leetcode-cn.com/problems/plus-one/)

>   给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。
>
>   最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
>   你可以假设除了整数 0 之外，这个整数不会以零开头。
>
>    
>
>   示例 1：
>
>   输入：digits = [1,2,3]
>   输出：[1,2,4]
>   解释：输入数组表示数字 123。
>   示例 2：
>
>   输入：digits = [4,3,2,1]
>   输出：[4,3,2,2]
>   解释：输入数组表示数字 4321。
>   示例 3：
>
>   输入：digits = [0]
>   输出：[1]
>
>
>   提示：
>
>   1 <= digits.length <= 100
>   0 <= digits[i] <= 9

```java
  public int[] plusOne(int[] digits) {
         for(int i = digits.length - 1; i >= 0; i --) {
            if(digits[i] != 9) {
                digits[i] ++;
                return digits;
            }
            
            digits[i] = 0;
        }
        
        int[] newDigits = new int[digits.length + 1];
        newDigits[0] = 1;
        return newDigits;
    }
```

#[剑指 Offer 26 - 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

>   输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)
>
>   B是A的子结构， 即 A中有出现和B相同的结构和节点值。
>
>   例如:
>   给定的树 A:
>
>        3
>       / \
>      4   5
>     / \
>    1   2
>   给定的树 B：
>
>      4 
>     /
>    1
>   返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。
>
>   示例 1：
>
>   输入：A = [1,2,3], B = [3,1]
>   输出：false
>   示例 2：
>
>   输入：A = [3,4,5,1,2], B = [4,1]
>   输出：true
>   限制：
>
>   0 <= 节点个数 <= 10000
>
>   通过次数55,700提交次数121,513
>

```java
 public boolean isSubStructure(TreeNode A, TreeNode B) {
        return (A != null && B != null) && (recur(A, B) 
        || isSubStructure(A.left, B)
        || isSubStructure(A.right, B));
    }
    boolean recur(TreeNode A, TreeNode B) {
        if(B == null) return true;
        if(A == null || A.val != B.val) return false;
        return recur(A.left, B.left) && recur(A.right, B.right);
    }
```

#[剑指 Offer 13 - 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

>   地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？
>
>    
>
>   示例 1：
>
>   输入：m = 2, n = 3, k = 1
>   输出：3
>   示例 2：
>
>   输入：m = 3, n = 1, k = 0
>   输出：1
>   提示：
>
>   1 <= n,m <= 100
>   0 <= k <= 20

```java
  public int movingCount(int m, int n, int k) {
        boolean[][] visited = new boolean[m][n];
        return dfs(0, 0, m, n, k, visited);
    }

    private int dfs(int i, int j, int m, int n, int k, boolean visited[][]) {
        if (i < 0 || i >= m || j < 0 || j >= n || (i/10 + i%10 + j/10 + j%10) > k || visited[i][j]) {
            return 0;
        }
        visited[i][j] = true;
        return dfs(i + 1, j, m, n, k, visited) + dfs(i - 1, j, m, n, k, visited) + 
               dfs(i, j + 1, m, n, k, visited) + dfs(i, j - 1, m, n, k, visited) + 1;
    }
```

#[面试题 01.01 - 判定字符是否唯一](https://leetcode-cn.com/problems/is-unique-lcci/)

>   实现一个算法，确定一个字符串 s 的所有字符是否全都不同。
>
>   示例 1：
>
>   输入: s = "leetcode"
>   输出: false 
>   示例 2：
>
>   输入: s = "abc"
>   输出: true
>   限制：
>
>   0 <= len(s) <= 100
>   如果你不使用额外的数据结构，会很加分。

```java
 public boolean isUnique(String astr) {
        int[] hash = new int[26];
        for(char ch : astr.toCharArray()) {
            hash[ch - 'a']++;
            if(hash[ch - 'a'] > 1) {
                return false;
            }
        }
        return true;
    }
```

#[剑指 Offer 56 - I - 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/) 

>   一个整型数组 nums 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。
>
>    
>
>   示例 1：
>
>   输入：nums = [4,1,4,6]
>   输出：[1,6] 或 [6,1]
>   示例 2：
>
>   输入：nums = [1,2,10,4,1,4,3,3]
>   输出：[2,10] 或 [10,2]
>
>
>   限制：
>
>   2 <= nums.length <= 10000

```java
 public int[] singleNumbers(int[] nums) {
        int sum = 0;
        for(int num : nums){
            sum ^= num;
        }
        int flag = sum & (-sum);
        int res = 0;
        for(int num : nums){
            if((flag & num) != 0){
                res ^= num;
            }
        }
        return new int[]{res,res ^ sum};
    }
```

#[260. 只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)

>   给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。
>
>   示例 :
>
>   输入: [1,2,1,3,2,5]
>   输出: [3,5]
>   注意：
>
>   结果输出的顺序并不重要，对于上面的例子， [5, 3] 也是正确答案。
>   你的算法应该具有线性时间复杂度。你能否仅使用常数空间复杂度来实现？

```java
public int[] singleNumber(int[] nums) {
        int num1 = 0, num2 = 0;
        int xor = 0;
        for(int num : nums)
            xor ^= num;
        
        int bit_1 = 1;
        while((xor & 1) == 0) {
            xor >>= 1;
            bit_1 <<= 1;
        }

        for(int num : nums) {
            if((num & bit_1) == 0)
                num1 ^= num;
            else
                num2 ^= num;
        }
        
        return new int[]{num1, num2};
    }
```

#[493. 翻转对](https://leetcode-cn.com/problems/reverse-pairs/) 

>   给定一个数组 nums ，如果 i < j 且 nums[i] > 2*nums[j] 我们就将 (i, j) 称作一个重要翻转对。
>
>   你需要返回给定数组中的重要翻转对的数量。
>
>   示例 1:
>
>   输入: [1,3,2,3,1]
>   输出: 2
>   示例 2:
>
>   输入: [2,4,3,5,1]
>   输出: 3
>   注意:
>
>   给定数组的长度不会超过50000。
>   输入数组中的所有数字都在32位整数的表示范围内。

```java
 public int reversePairs(int[] nums) {
        if (nums == null || nums.length < 2) return 0;
        int n = nums.length;
        return mergeSort(nums, Arrays.copyOf(nums, n), 0, n - 1);
    }

    private int mergeSort(int[] nums, int[] tmps, int lo, int hi) {
        if (lo >= hi) return 0;
        int mid = (lo + hi) >> 1;
        int lc = mergeSort(tmps, nums, lo, mid);
        int rc = mergeSort(tmps, nums, mid + 1, hi);
        int i = lo, j = mid + 1, cnt = 0;
        while (i <= mid && j <= hi) {
            if (tmps[i] > 2.0 * tmps[j]) {
                cnt += mid - i + 1;
                ++j;
            } else ++i;
        }
        merge(nums, tmps, lo, hi);
        return lc + rc + cnt;
    }

    private void merge(int[] nums, int[] tmps, int lo, int hi) {
        if (lo >= hi) return;
        int mid = (lo + hi) >> 1;
        int i = lo, j = mid + 1, pos = lo;
        while (i <= mid && j <= hi)
            nums[pos++] = tmps[i] < tmps[j] ? tmps[i++] : tmps[j++];
        while (i <= mid) nums[pos++] = tmps[i++];
        while (j <= hi) nums[pos++] = tmps[j++];
    }
```

# [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

>   根据一棵树的中序遍历与后序遍历构造二叉树。
>
>   注意:
>   你可以假设树中没有重复的元素。
>
>   例如，给出
>
>   中序遍历 inorder = [9,3,15,20,7]
>   后序遍历 postorder = [9,15,7,20,3]
>   返回如下的二叉树：
>
>   ```java
>   		3
>      / \
>     9  20
>       /  \
>      15   7
>   ```

```java
 public TreeNode buildTree(int[] inorder, int[] postorder) {
         return helper(inorder, postorder, postorder.length - 1, 0, inorder.length - 1);
    }
      public TreeNode helper(int[] inorder, int[] postorder, int postEnd, int inStart, int inEnd) {
        if (inStart > inEnd) {
            return null;
        }

        int currentVal = postorder[postEnd];
        TreeNode current = new TreeNode(currentVal);
        
        int inIndex = 0; 
        for (int i = inStart; i <= inEnd; i++) {
            if (inorder[i] == currentVal) {
                inIndex = i;
            }
        }
        TreeNode left = helper(inorder, postorder, postEnd - (inEnd- inIndex) - 1, inStart, inIndex - 1);
        TreeNode right = helper(inorder, postorder, postEnd - 1, inIndex + 1, inEnd);
        current.left = left;
        current.right = right;
        return current;
    }
```

#[100. 相同的树](https://leetcode-cn.com/problems/same-tree/) 

>   

```java

```

#[125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/) 

>   

```java

```

# [95. 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

>   给定一个整数 n，生成所有由 1 ... n 为节点所组成的 二叉搜索树 。
>
>    
>
>   示例：
>
>   输入：3
>   输出：
>   [
>     [1,null,3,2],
>     [3,2,null,1],
>     [3,1,null,null,2],
>     [2,1,3],
>     [1,null,2,null,3]
>   ]
>   解释：
>   以上的输出对应以下 5 种不同结构的二叉搜索树：
>
>      1         3     3      2      1
>       \       /     /      / \      \
>        3     2     1      1   3      2
>       /     /       \                 \
>      2     1         2                 3

```java
 public List<TreeNode> generateTrees(int n) {
        if(n==0)return Collections.emptyList();
        return findTrees(1,n);
    }
    public List<TreeNode> findTrees(int start,int end){
        List<TreeNode> all = new ArrayList<>();
        if(start > end){
            all.add(null);
            return all;
        }
        for(int i = start;i <= end;i++){
            List<TreeNode> left = findTrees(start,i-1);
            List<TreeNode> right = findTrees(i+1,end);
            for(TreeNode l : left){
                for(TreeNode r: right){
                    TreeNode cur = new TreeNode(i);
                    cur.left = l;
                    cur.right = r;
                    all.add(cur);
                }
            }
        }
        return all;
    }
```

# [437. 路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

>   给定一个二叉树，它的每个结点都存放着一个整数值。
>
>   找出路径和等于给定数值的路径总数。
>
>   路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。
>
>   二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。
>
>   示例：
>
>   root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8
>
>         10
>        /  \
>       5   -3
>      / \    \
>     3   2   11
>    / \   \
>   3  -2   1
>
>   返回 3。和等于 8 的路径有:
>
>   1.  5 -> 3
>   2.  5 -> 2 -> 1
>   3.  -3 -> 11
>

```java
   public int pathSum(TreeNode root, int sum) {
       if(root == null) return 0;
       return getSum(root,sum,new int[1000],0);
    }

    public int getSum(TreeNode root, int sum, int[] array, int curr){
        if(root == null) return 0;
        array[curr] = root.val;
        int count = 0, thisSum = sum;
        for(int i = curr; i >= 0; i--){
            thisSum -= array[i];
            if(thisSum == 0) count++;
        }
        return count + getSum(root.left,sum,array,curr+1) + getSum(root.right,sum,array,curr+1);
    }
```

#[392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/) 

>   给定字符串 s 和 t ，判断 s 是否为 t 的子序列。
>
>   你可以认为 s 和 t 中仅包含英文小写字母。字符串 t 可能会很长（长度 ~= 500,000），而 s 是个短字符串（长度 <=100）。
>
>   字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，"ace"是"abcde"的一个子序列，而"aec"不是）。
>
>   示例 1:
>   s = "abc", t = "ahbgdc"
>
>   返回 true.
>
>   示例 2:
>   s = "axc", t = "ahbgdc"
>
>   返回 false.
>
>   后续挑战 :
>
>   如果有大量输入的 S，称作S1, S2, ... , Sk 其中 k >= 10亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？
>

```java
 public boolean isSubsequence(String s, String t) {
        int n = s.length(), m = t.length();
        int i = 0, j = 0;
        while (i < n && j < m) {
            if (s.charAt(i) == t.charAt(j)) {
                i++;
            }
            j++;
        }
        return i == n;
    }
```

# [312. 戳气球](https://leetcode-cn.com/problems/burst-balloons/)

>   有 n 个气球，编号为0 到 n-1，每个气球上都标有一个数字，这些数字存在数组 nums 中。
>
>   现在要求你戳破所有的气球。如果你戳破气球 i ，就可以获得 nums[left] * nums[i] * nums[right] 个硬币。 这里的 left 和 right 代表和 i 相邻的两个气球的序号。注意当你戳破了气球 i 后，气球 left 和气球 right 就变成了相邻的气球。
>
>   求所能获得硬币的最大数量。
>
>   说明:
>
>   你可以假设 nums[-1] = nums[n] = 1，但注意它们不是真实存在的所以并不能被戳破。
>   0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100
>   示例:
>
>   输入: [3,1,5,8]
>   输出: 167 
>   解释: nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
>        coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167

```java
 public int maxCoins(int[] nums) {
        int n = nums.length;
        int[][] rec = new int[n + 2][n + 2];
        int[] val = new int[n + 2];
        val[0] = val[n + 1] = 1;
        for (int i = 1; i <= n; i++) {
            val[i] = nums[i - 1];
        }
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i + 2; j <= n + 1; j++) {
                for (int k = i + 1; k < j; k++) {
                    int sum = val[i] * val[k] * val[j];
                    sum += rec[i][k] + rec[k][j];
                    rec[i][j] = Math.max(rec[i][j], sum);
                }
            }
        }
        return rec[0][n + 1];
    }
```

#[679. 24 点游戏](https://leetcode-cn.com/problems/24-game/) 

>   
>   你有 4 张写有 1 到 9 数字的牌。你需要判断是否能通过 `*`，`/`，`+`，`-`，`(`，`)` 的运算得到 24。
>
>   **示例 1:**
>
>   ```
>   输入: [4, 1, 8, 7]
>   输出: True
>   解释: (8-4) * (7-1) = 24
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: [1, 2, 1, 2]
>   输出: False
>   ```
>
>   **注意:**
>
>   1. 除法运算符 `/` 表示实数除法，而不是整数除法。例如 4 / (1 - 2/3) = 12 。
>   2. 每个运算符对两个数进行运算。特别是我们不能用 `-` 作为一元运算符。例如，`[1, 1, 1, 1]` 作为输入时，表达式 `-1 - 1 - 1 - 1` 是不允许的。
>   3. 你不能将数字连接在一起。例如，输入为 `[1, 2, 1, 2]` 时，不能写成 12 + 12 。

```java
static final int TARGET = 24;
    static final double EPSILON = 1e-6;
    static final int ADD = 0, MULTIPLY = 1, SUBTRACT = 2, DIVIDE = 3;

    public boolean judgePoint24(int[] nums) {
        List<Double> list = new ArrayList<Double>();
        for (int num : nums) {
            list.add((double) num);
        }
        return solve(list);
    }

    public boolean solve(List<Double> list) {
        if (list.size() == 0) {
            return false;
        }
        if (list.size() == 1) {
            return Math.abs(list.get(0) - TARGET) < EPSILON;
        }
        int size = list.size();
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                if (i != j) {
                    List<Double> list2 = new ArrayList<Double>();
                    for (int k = 0; k < size; k++) {
                        if (k != i && k != j) {
                            list2.add(list.get(k));
                        }
                    }
                    for (int k = 0; k < 4; k++) {
                        if (k < 2 && i > j) {
                            continue;
                        }
                        if (k == ADD) {
                            list2.add(list.get(i) + list.get(j));
                        } else if (k == MULTIPLY) {
                            list2.add(list.get(i) * list.get(j));
                        } else if (k == SUBTRACT) {
                            list2.add(list.get(i) - list.get(j));
                        } else if (k == DIVIDE) {
                            if (Math.abs(list.get(j)) < EPSILON) {
                                continue;
                            } else {
                                list2.add(list.get(i) / list.get(j));
                            }
                        }
                        if (solve(list2)) {
                            return true;
                        }
                        list2.remove(list2.size() - 1);
                    }
                }
            }
        }
        return false;
    }
```

#[378. 有序矩阵中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/)

>   给定一个 n x n 矩阵，其中每行和每列元素均按升序排序，找到矩阵中第 k 小的元素。
>   请注意，它是排序后的第 k 小元素，而不是第 k 个不同的元素。
>
>   示例：
>
>   ```sql
>   matrix = [
>      [ 1,  5,  9],
>      [10, 11, 13],
>      [12, 13, 15]
>   ],
>   k = 8,
>   
>   返回 13。
>   ```
>
>   提示：
>   你可以假设 k 的值永远是有效的，1 ≤ k ≤ n2 。

```java
public int kthSmallest(int[][] matrix, int k) {
        int m = matrix.length, n = matrix[0].length;
        int lo = matrix[0][0], hi = matrix[m - 1][n - 1];
        while (lo <= hi) {
            int cnt = 0, mid = lo + (hi - lo) / 2;
            int i = m - 1, j = 0;
            while (i >= 0 && j < n) {
                if (matrix[i][j] <= mid) {
                    cnt += i + 1;
                    j++;
                } else i--;
            }
            if (cnt < k) lo = mid + 1;
            else hi = mid - 1;
        }
        return lo;
    }
```

### 柱状图中的最大矩形执行流程

![image-20201124191215663](/Users/lumac/Library/Application Support/typora-user-images/image-20201124191215663.png)

我们想一下最好计算的情况肯定是递减的,

假设只有个2,1

2,1的执行流程呢

如果是2,1,2呢

i = 0

​	stack.peek = -1 stack.push(0) stack[-1,0]

i = 1

​	前一个高度是大于当前的高度,不进入while循环

​	stack.push(1) stack[-1,0,1]

i = 2

​	前一个高度是1,当前的高度2,进入while循环

​	max = 2*(2-)



**算法总结**:

1.如果当前的高度小于栈里保存的第一个的高度

2.



**执行流程**

```java
public static int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(-1);
    int maxarea = 0;
    for (int i = 0; i < heights.length; ++i) {
        while (stack.peek() != -1 && heights[stack.peek()] >= heights[i]) {
            maxarea = Math.max(maxarea, heights[stack.pop()] * (i - stack.peek() - 1));
        }
        stack.push(i);
    }
    while (stack.peek() != -1)
        maxarea = Math.max(maxarea, heights[stack.pop()] * (heights.length - stack.peek() - 1));
    return maxarea;
}
```
