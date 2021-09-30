[toc]

#[204. 计数质数](https://leetcode-cn.com/problems/count-primes/) 

>  统计所有小于非负整数 n 的质数的数量。
>
>   
>
>  示例 1：
>
>  输入：n = 10
>  输出：4
>  解释：小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。
>  示例 2：
>
>  输入：n = 0
>  输出：0
>  示例 3：
>
>  输入：n = 1
>  输出：0
>
>
>  提示：
>
>  0 <= n <= 5 * 106
>

```java
public int countPrimes(int n) {
        boolean[] isPrime = new boolean[n + 1];
        int count = 0;
        for (int i = 2; i < n; i++) {
            if (isPrime[i] == false) {
                count++;
                for (int j = i + i; j < n; j += i) {
                    isPrime[j] = true;
                }
            }
        }
        return count;
    }
```



#[57. 插入区间](https://leetcode-cn.com/problems/insert-interval/) 

>  给出一个无重叠的 ，按照区间起始端点排序的区间列表。
>
>  在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。
>
>   
>
>  示例 1：
>
>  输入：intervals = [[1,3],[6,9]], newInterval = [2,5]
>  输出：[[1,5],[6,9]]
>  示例 2：
>
>  输入：intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
>  输出：[[1,2],[3,10],[12,16]]
>  解释：这是因为新的区间 [4,8] 与 [3,5],[6,7],[8,10] 重叠。
>
>
>  注意：输入类型已在 2019 年 4 月 15 日更改。请重置为默认代码定义以获取新的方法签名。
>

```java
 public int[][] insert(int[][] intervals, int[] newInterval) {
    int len = intervals.length;
            // 左侧最靠近新区间并且无重叠的区间下标
            int left = -1;
            while (left + 1 < len && intervals[left + 1][1] < newInterval[0]) {
                left++;
            }
            // 右侧最靠近新区间并且无重叠的区间下标
            int right = len;
            while (right - 1 >= 0 && intervals[right - 1][0] > newInterval[1]) {
                right--;
            }
            // 新区间下标
            int index = left + 1;
            int[] merged;
            if (left + 1 == right) {
                merged = newInterval;
            } else {
                merged = new int[]{Math.min(intervals[left + 1][0], newInterval[0]), Math.max(intervals[right - 1][1], newInterval[1])};
            }
            int[][] ans = new int[left + 1 + len - right + 1][];
            System.arraycopy(intervals, 0, ans, 0, left + 1);
            ans[index] = merged;
            System.arraycopy(intervals, right, ans, index + 1, len - right);
            return ans;
    }
```



#[421. 数组中两个数的最大异或值](https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/) 有趣

>  
>  给定一个非空数组，数组中元素为 a0, a1, a2, … , an-1，其中 0 ≤ ai < 231 。
>
>  找到 ai 和aj 最大的异或 (XOR) 运算结果，其中0 ≤ *i*, *j* < *n* 。
>
>  你能在O(*n*)的时间解决这个问题吗？
>
>  **示例:**
>
>  ```
>  输入: [3, 10, 5, 25, 2, 8]
>  
>  输出: 28
>  
>  解释: 最大的结果是 5 ^ 25 = 28.
>  ```

```java
 static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;
        TreeNode(int x) { val = x; }
    }
    
    public int findMaximumXOR(int[] nums) {
        TreeNode root = new TreeNode(-1);
        
        // build the tree
        for(int n : nums) {
            TreeNode node = root;
            for(int i = 31; i>=0; i--) {
                if ((n & (1 << i)) == 0) { // 0
                    if (node.left == null) {
                        node.left = new TreeNode(0);
                    }
                    node = node.left;
                } else { // 1
                    if (node.right == null) {
                        node.right = new TreeNode(1);
                    }
                    node = node.right;
                }
            }
            node.left = new TreeNode(n);
        }
        
        int max = 0;
        for(int n: nums) {
            TreeNode node = root;
            for(int i=31; i>=0; i--) {
                if ((n & (1<<i)) == 0) {
                    if (node.right != null) {
                        node = node.right;
                    } else {
                        node = node.left;
                    }
                } else {
                    if (node.left != null) {
                        node = node.left;
                    } else {
                        node = node.right;
                    }
                }
            }
            int nn = node.left.val;
            max = Math.max(max, n ^ nn);
        }
        return max;
    }
```

# [剑指 Offer 68 - II - 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

>  给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
>  百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”
>
>  例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]
>
>  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/binarytree.png) 
>
>  示例 1:
>
>  输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
>  输出: 3
>  解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
>  示例 2:
>
>  输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
>  输出: 5
>  解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。
>
>
>  说明:
>
>  所有节点的值都是唯一的。
>  p、q 为不同节点且均存在于给定的二叉树中。
>  注意：本题与主站 236 题相同：https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/

```java
  public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        //记录遍历到的每个节点的父节点。
        Map<TreeNode, TreeNode> parent = new HashMap<>();
        Queue<TreeNode> queue = new LinkedList<>();
        parent.put(root, null);//根节点没有父节点，所以为空
        queue.add(root);
        //直到两个节点都找到为止。
        while (!parent.containsKey(p) || !parent.containsKey(q)) {
            //队列是一边进一边出，这里poll方法是出队，
            TreeNode node = queue.poll();
            if (node.left != null) {
                //左子节点不为空，记录下他的父节点
                parent.put(node.left, node);
                //左子节点不为空，把它加入到队列中
                queue.add(node.left);
            }
            //右节点同上
            if (node.right != null) {
                parent.put(node.right, node);
                queue.add(node.right);
            }
        }
        Set<TreeNode> ancestors = new HashSet<>();
        //记录下p和他的祖先节点，从p节点开始一直到根节点。
        while (p != null) {
            ancestors.add(p);
            p = parent.get(p);
        }
        //查看p和他的祖先节点是否包含q节点，如果不包含再看是否包含q的父节点……
        while (!ancestors.contains(q))
            q = parent.get(q);
        return q;
    }
```

```java
 public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null) {
            return null;
        }
        if (root == p || root == q) {
            return root;
        }
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left != null && right != null) {
            // p q 一个在左，一个在右
            return root;
        }
        if (left != null) {
            // p q 都在左子树
            return left;
        }
        if (right != null) {
            // p q 都在右子树
            return right;
        }
        return null;
    }
```



#[剑指 Offer 58 - II - 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

>  字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。
>
>   
>
>  示例 1：
>
>  输入: s = "abcdefg", k = 2
>  输出: "cdefgab"
>  示例 2：
>
>  输入: s = "lrloseumgh", k = 6
>  输出: "umghlrlose"
>
>
>  限制：
>
>  1 <= k < s.length <= 10000

```java
  public String reverseLeftWords(String s, int n) {
        if(s == null || s.length() == 0) return s;
        if(n >s.length()) return s;
        String pre = s.substring(0,n);
        String last = s.substring(n,s.length());
        return last+pre;
    }
```



#[剑指 Offer 57 - 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/) 

>  输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。
>
>   
>
>  示例 1：
>
>  输入：nums = [2,7,11,15], target = 9
>  输出：[2,7] 或者 [7,2]
>  示例 2：
>
>  输入：nums = [10,26,30,31,47,60], target = 40
>  输出：[10,30] 或者 [30,10]
>
>
>  限制：
>
>  1 <= nums.length <= 10^5
>  1 <= nums[i] <= 10^6

```java
     int len = nums.length;
        int left = 0;
        int right = len - 1;
        int[] res = new int[2];
        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) {
                res[0] = nums[left];
                res[1] = nums[right];
                return res;
            }
            if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
        return new int[0];
    }
```



#[132. 分割回文串 II](https://leetcode-cn.com/problems/palindrome-partitioning-ii/) 

>  
>  给定一个字符串 *s*，将 *s* 分割成一些子串，使每个子串都是回文串。
>
>  返回符合要求的最少分割次数。
>
>  **示例:**
>
>  ```
>  输入: "aab"
>  输出: 1
>  解释: 进行一次分割就可将 s 分割成 ["aa","b"] 这样两个回文子串。
>  ```

```java
  public int minCut(String s) {
        if(s == null || s.length() <= 1)
            return 0;
        int len = s.length();
        int dp[] = new int[len];
        Arrays.fill(dp, len-1);
        for(int i = 0; i < len; i++){
            // 注意偶数长度与奇数长度回文串的特点
            mincutHelper(s , i , i , dp);  // 奇数回文串以1个字符为中心
            mincutHelper(s, i , i+1 , dp); // 偶数回文串以2个字符为中心
        }
        return dp[len-1];
    }
    private void mincutHelper(String s, int i, int j, int[] dp){
        int len = s.length();
        while(i >= 0 && j < len && s.charAt(i) == s.charAt(j)){
            dp[j] = Math.min(dp[j] , (i==0?-1:dp[i-1])+1);
            i--;
            j++;
        }
    }
```



#[剑指 Offer 50 - 第一个只出现一次的字](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/) 

>  在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。
>
>  示例:
>
>  s = "abaccdeff"
>  返回 "b"
>
>  s = "" 
>  返回 " "
>
>
>  限制：
>
>  0 <= s 的长度 <= 50000
>

```java
  public char firstUniqChar(String s) {
        int[] count = new int[256];
        char[] chars = s.toCharArray();
        for(char c : chars)
            count[c]++;
        for(char c : chars){
            if(count[c] == 1)
                return c;
        }
        return ' ';
    }
```



#[剑指 Offer 20 - 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/) 

>  请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。
>

```java
public boolean isNumber(String s) {
        Map<State, Map<CharType, State>> transfer = new HashMap<State, Map<CharType, State>>();
        Map<CharType, State> initialMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_SPACE, State.STATE_INITIAL);
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
            put(CharType.CHAR_SIGN, State.STATE_INT_SIGN);
        }};
        transfer.put(State.STATE_INITIAL, initialMap);
        Map<CharType, State> intSignMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
        }};
        transfer.put(State.STATE_INT_SIGN, intSignMap);
        Map<CharType, State> integerMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_POINT, State.STATE_POINT);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_INTEGER, integerMap);
        Map<CharType, State> pointMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_POINT, pointMap);
        Map<CharType, State> pointWithoutIntMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
        }};
        transfer.put(State.STATE_POINT_WITHOUT_INT, pointWithoutIntMap);
        Map<CharType, State> fractionMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
            put(CharType.CHAR_EXP, State.STATE_EXP);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_FRACTION, fractionMap);
        Map<CharType, State> expMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
            put(CharType.CHAR_SIGN, State.STATE_EXP_SIGN);
        }};
        transfer.put(State.STATE_EXP, expMap);
        Map<CharType, State> expSignMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
        }};
        transfer.put(State.STATE_EXP_SIGN, expSignMap);
        Map<CharType, State> expNumberMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_EXP_NUMBER, expNumberMap);
        Map<CharType, State> endMap = new HashMap<CharType, State>() {{
            put(CharType.CHAR_SPACE, State.STATE_END);
        }};
        transfer.put(State.STATE_END, endMap);

        int length = s.length();
        State state = State.STATE_INITIAL;

        for (int i = 0; i < length; i++) {
            CharType type = toCharType(s.charAt(i));
            if (!transfer.get(state).containsKey(type)) {
                return false;
            } else {
                state = transfer.get(state).get(type);
            }
        }
        return state == State.STATE_INTEGER || state == State.STATE_POINT || state == State.STATE_FRACTION || state == State.STATE_EXP_NUMBER || state == State.STATE_END;
    }

    public CharType toCharType(char ch) {
        if (ch >= '0' && ch <= '9') {
            return CharType.CHAR_NUMBER;
        } else if (ch == 'e' || ch == 'E') {
            return CharType.CHAR_EXP;
        } else if (ch == '.') {
            return CharType.CHAR_POINT;
        } else if (ch == '+' || ch == '-') {
            return CharType.CHAR_SIGN;
        } else if (ch == ' ') {
            return CharType.CHAR_SPACE;
        } else {
            return CharType.CHAR_ILLEGAL;
        }
    }

    enum State {
        STATE_INITIAL,
        STATE_INT_SIGN,
        STATE_INTEGER,
        STATE_POINT,
        STATE_POINT_WITHOUT_INT,
        STATE_FRACTION,
        STATE_EXP,
        STATE_EXP_SIGN,
        STATE_EXP_NUMBER,
        STATE_END,
    }

    enum CharType {
        CHAR_NUMBER,
        CHAR_EXP,
        CHAR_POINT,
        CHAR_SIGN,
        CHAR_SPACE,
        CHAR_ILLEGAL,
    }
```



#[剑指 Offer 64 - 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/) 

>  求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。
>
>   
>
>  示例 1：
>
>  输入: n = 3
>  输出: 6
>  示例 2：
>
>  输入: n = 9
>  输出: 45
>
>
>  限制：
>
>  1 <= n <= 10000

```java
  public int sumNums(int n) {
        boolean flag = (n > 0) && (n += sumNums(n -1)) > 0;
        return n;
    }
```



#[974. 和可被 K 整除的子数组](https://leetcode-cn.com/problems/subarray-sums-divisible-by-k/) 

>  给定一个整数数组 A，返回其中元素之和可被 K 整除的（连续、非空）子数组的数目。
>
>   
>
>  示例：
>
>  输入：A = [4,5,0,-2,-3,1], K = 5
>  输出：7
>  解释：
>  有 7 个子数组满足其元素之和可被 K = 5 整除：
>  [4, 5, 0, -2, -3, 1], [5], [5, 0], [5, 0, -2, -3], [0], [0, -2, -3], [-2, -3]
>
>
>  提示：
>
>  1 <= A.length <= 30000
>  -10000 <= A[i] <= 10000
>  2 <= K <= 10000

```java
  public int subarraysDivByK(int[] A, int k) {
       int[] m = new int[k];
        m[0] = 1;
        int count = 0,sum = 0;
        for (int a: A ) {
            sum = (sum+a)%k;
            if(sum<0)
            {
                sum +=k;
            }
            count +=m[sum];
            m[sum]++;
        }
        return count;
    }
```



#[449. 序列化和反序列化二叉搜索树](https://leetcode-cn.com/problems/serialize-and-deserialize-bst/) 

```java
 // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        return core(root);
    }

    private String core(TreeNode root) {
        if (root == null)   return "#,";
        String s = root.val + "," ;
        s += core(root.left);
        s += core(root.right);
        return s;
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if (data.length() == 0) return null;
        String[] values = data.split(",");
        int[] ints = new int[values.length];
        for (int i = 0; i < values.length; i++) {
            if (values[i].equals("#")) {
                ints[i] = -1; continue;
            }
            ints[i] = Integer.parseInt(values[i]);
        }
        return build(ints, 0, ints.length - 1);
    }

    private TreeNode build(int[] preOrder, int preStart, int preEnd) {
        TreeNode root = newNode(preOrder[preStart]);
        if (root == null ||preStart == preEnd) {
            return root;
        }
        // 找到左子树的结束点
        int leftEnd = preStart;
        while (leftEnd < preEnd && (preOrder[leftEnd] == -1 ||  preOrder[leftEnd] <= root.val))
            leftEnd ++;

        root.left = build(preOrder, preStart + 1, leftEnd - 1);
        root.right = build(preOrder, leftEnd, preEnd);
        return root;
    }

    private TreeNode newNode(int val) {
        if (val == -1) return null;
        return new TreeNode(val);
    }
```



#[1480. 一维数组的动态和](https://leetcode-cn.com/problems/running-sum-of-1d-array/) 

>  给你一个数组 nums 。数组「动态和」的计算公式为：runningSum[i] = sum(nums[0]…nums[i]) 。
>
>  请返回 nums 的动态和。
>
>   
>
>  示例 1：
>
>  输入：nums = [1,2,3,4]
>  输出：[1,3,6,10]
>  解释：动态和计算过程为 [1, 1+2, 1+2+3, 1+2+3+4] 。
>  示例 2：
>
>  输入：nums = [1,1,1,1,1]
>  输出：[1,2,3,4,5]
>  解释：动态和计算过程为 [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1] 。
>  示例 3：
>
>  输入：nums = [3,1,2,10,1]
>  输出：[3,4,6,16,17]
>
>
>  提示：
>
>  1 <= nums.length <= 1000
>  -10^6 <= nums[i] <= 10^6

```java
 public int[] runningSum(int[] nums) {
        int[] res = new int[nums.length];
        if(nums==null||nums.length ==0)return res;
        Map<Integer,Integer> cache = new HashMap<>();
        for(int i = 0;i<nums.length;i++){
            cache.put(i,cache.getOrDefault(i-1,0)+nums[i]);
        }
        for(int i = 0;i<nums.length;i++){
            res[i] = cache.get(i);
        }
        return res;
    }
```



#[129. 求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/) 

>  给定一个二叉树，它的每个结点都存放一个 0-9 的数字，每条从根到叶子节点的路径都代表一个数字。
>
>  例如，从根到叶子节点路径 1->2->3 代表数字 123。
>
>  计算从根到叶子节点生成的所有数字之和。
>
>  说明: 叶子节点是指没有子节点的节点。
>
>  示例 1:
>
>  输入: [1,2,3]
>      1
>     / \
>    2   3
>  输出: 25
>  解释:
>  从根到叶子节点路径 1->2 代表数字 12.
>  从根到叶子节点路径 1->3 代表数字 13.
>  因此，数字总和 = 12 + 13 = 25.
>  示例 2:
>
>  输入: [4,9,0,5,1]
>      4
>     / \
>    9   0
>   / \
>  5   1
>  输出: 1026
>  解释:
>  从根到叶子节点路径 4->9->5 代表数字 495.
>  从根到叶子节点路径 4->9->1 代表数字 491.
>  从根到叶子节点路径 4->0 代表数字 40.
>  因此，数字总和 = 495 + 491 + 40 = 1026.

```java
int sum = 0;
    public int sumNumbers(TreeNode root) {
        helper(root, 0);
        return sum;
    }
    
    public void helper(TreeNode root, int pre) {
        if(root == null) return;
        if(root.left == null && root.right == null){
            sum += pre * 10  + root.val;
        } else{
            pre = pre * 10 + root.val;
            helper(root.left, pre);
            helper(root.right, pre);
        }
    }
```



#[572. 另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/) 

>  给定两个非空二叉树 s 和 t，检验 s 中是否包含和 t 具有相同结构和节点值的子树。s 的一个子树包括 s 的一个节点和这个节点的所有子孙。s 也可以看做它自身的一棵子树。
>

```java
public boolean isSubtree(TreeNode s, TreeNode t) {
        if(s==null||t==null)
            return false;
        if(s.val==t.val&&sameRoot(s,t))
            return true;
        return isSubtree(s.left,t)||isSubtree(s.right,t);
    }

    public boolean sameRoot(TreeNode r1,TreeNode r2){
        if(r1==null&&r2==null)
            return true;
        if(r1==null||r2==null)
            return false;
        if(r1.val==r2.val)
            return sameRoot(r1.left,r2.left)&&sameRoot(r1.right,r2.right);
        return false;
    }
```



#[133. 克隆图](https://leetcode-cn.com/problems/clone-graph/) 

>  给你无向 连通 图中一个节点的引用，请你返回该图的 深拷贝（克隆）。
>
>  图中的每个节点都包含它的值 val（int） 和其邻居的列表（list[Node]）。
>
>  class Node {
>      public int val;
>      public List<Node> neighbors;
>  }
>
>
>  测试用例格式：
>
>  简单起见，每个节点的值都和它的索引相同。例如，第一个节点值为 1（val = 1），第二个节点值为 2（val = 2），以此类推。该图在测试用例中使用邻接列表表示。
>
>  邻接列表 是用于表示有限图的无序列表的集合。每个列表都描述了图中节点的邻居集。
>
>  给定节点将始终是图中的第一个节点（值为 1）。你必须将 给定节点的拷贝 作为对克隆图的引用返回。
>

```java
 private HashMap <Node, Node> visited = new HashMap <> ();
    public Node cloneGraph(Node node) {
        if (node == null) {
            return node;
        }

        // If the node was already visited before.
        // Return the clone from the visited dictionary.
        if (visited.containsKey(node)) {
            return visited.get(node);
        }

        // Create a clone for the given node.
        // Note that we don't have cloned neighbors as of now, hence [].
        Node cloneNode = new Node(node.val, new ArrayList());
        // The key is original node and value being the clone node.
        visited.put(node, cloneNode);

        // Iterate through the neighbors to generate their clones
        // and prepare a list of cloned neighbors to be added to the cloned node.
        for (Node neighbor: node.neighbors) {
            cloneNode.neighbors.add(cloneGraph(neighbor));
        }
        return cloneNode;
        }
```



#[463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/) 

>  给定一个 row x col 的二维网格地图 grid ，其中：grid[i][j] = 1 表示陆地， grid[i][j] = 0 表示水域。
>
>  网格中的格子 水平和垂直 方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。
>
>  岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。
>
>   
>
>  示例 1：
>
>  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/island.png)
>
>  输入：grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]]
>  输出：16
>  解释：它的周长是上面图片中的 16 个黄色的边
>  示例 2：
>
>  输入：grid = [[1]]
>  输出：4
>  示例 3：
>
>  输入：grid = [[1,0]]
>  输出：4

```java
      //二维数组的宽度
        int len1 = grid.length;
        //二维数组的长度
        int len2 = grid[0].length;
        //周长初始化为0
        int perimeter = 0;
        //开始遍历
        for (int i = 0; i < len1; i++) {
            for (int j = 0; j < len2; j++) {
                if (grid[i][j] == 0) {
                    continue;
                }
                if (i - 1 < 0 || grid[i - 1][j] == 0) {//上边
                    perimeter++;
                }
                if (j - 1 < 0 || grid[i][j - 1] == 0) {//左边
                    perimeter++;
                }
                if (i + 1 == len1 || i + 1 < len1 && grid[i + 1][j] == 0) {
                    perimeter++;
                }
                if (j + 1 == len2 || j + 1 < len2 && grid[i][j + 1] == 0) {
                    perimeter++;
                }
            }
        }
        return perimeter;
    }
```



#[1004. 最大连续1的个数 III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/) 

>  给定一个二进制数组， 计算其中最大连续1的个数。
>
>  示例 1:
>
>  输入: [1,1,0,1,1,1]
>  输出: 3
>  解释: 开头的两位和最后的三位都是连续1，所以最大连续1的个数是 3.
>  注意：
>
>  输入的数组只包含 0 和1。
>  输入数组的长度是正整数，且不超过 10,000。

```java
//简洁优雅 
public int findMaxConsecutiveOnes(int[] nums) {
        int cnt = 0, max = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 0) {
                cnt = 0;
            } else if (++cnt > max) {
                max = cnt;
            }
        }
        return max;
    }
```



#[1014. 最佳观光组合](https://leetcode-cn.com/problems/best-sightseeing-pair/) 

>  给定正整数数组 A，A[i] 表示第 i 个观光景点的评分，并且两个景点 i 和 j 之间的距离为 j - i。
>
>  一对景点（i < j）组成的观光组合的得分为（A[i] + A[j] + i - j）：景点的评分之和减去它们两者之间的距离。
>
>  返回一对观光景点能取得的最高分。
>
>   
>
>  示例：
>
>  输入：[8,1,5,2,6]
>  输出：11
>  解释：i = 0, j = 2, A[i] + A[j] + i - j = 8 + 5 + 0 - 2 = 11
>
>
>  提示：
>
>  2 <= A.length <= 50000
>  1 <= A[i] <= 1000

```java
 public int maxScoreSightseeingPair(int[] A) {
        int left = A[0],res = Integer.MIN_VALUE;
        for(int i = 1;i<A.length;i++){
            res = Math.max(res,A[i]-i+left);
            left = Math.max(left,A[i]+i);
        }
        return res;
    }
```



#[455. 分发饼干](https://leetcode-cn.com/problems/assign-cookies/) 

>  假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。
>
>  对每个孩子 i，都有一个胃口值 g[i]，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干 j，都有一个尺寸 s[j] 。如果 s[j] >= g[i]，我们可以将这个饼干 j 分配给孩子 i ，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。
>
>
>  示例 1:
>
>  输入: g = [1,2,3], s = [1,1]
>  输出: 1
>  解释: 
>  你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。
>  虽然你有两块小饼干，由于他们的尺寸都是1，你只能让胃口值是1的孩子满足。
>  所以你应该输出1。
>  示例 2:
>
>  输入: g = [1,2], s = [1,2,3]
>  输出: 2
>  解释: 
>  你有两个孩子和三块小饼干，2个孩子的胃口值分别是1,2。
>  你拥有的饼干数量和尺寸都足以让所有孩子满足。
>  所以你应该输出2.
>
>
>  提示：
>
>  1 <= g.length <= 3 * 104
>  0 <= s.length <= 3 * 104
>  1 <= g[i], s[j] <= 231 - 1

```java
public int findContentChildren(int[] g, int[] s) {
       int res = 0, index = 0;
        Arrays.sort(g);
        Arrays.sort(s);
        for (int value : s) {
            if (value >= g[index]) {
                res++;
                index++;
                if (index >= g.length) {
                    break;
                }
            }
        }
        return res;
    }
```

