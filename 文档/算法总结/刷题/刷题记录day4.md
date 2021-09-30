[toc]

# [221. 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

>   
>   在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。
>
>    
>
>   **示例：**
>
>   ```java
>   输入：
>   matrix = [["1","0","1","0","0"],
>             ["1","0","1","1","1"],
>             ["1","1","1","1","1"],
>             ["1","0","0","1","0"]]
>   
>   输出：4
>   ```
>
>   时间复杂度O(m*n)
>
>   空间复杂度O(m*n)

```java
 public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[] dp = new int[cols + 1];
        int maxsqlen = 0, prev = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                int temp = dp[j];
                if (matrix[i - 1][j - 1] == '1') {
                    dp[j] = Math.min(Math.min(dp[j - 1], prev), dp[j]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[j]);
                } else {
                    dp[j] = 0;
                }
                prev = temp;
            }
        }
        return maxsqlen * maxsqlen;
    }
```

# [剑指 Offer 03 - 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

>   找出数组中重复的数字。
>
>
>   在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。
>
>   示例 1：
>
>   输入：
>   [2, 3, 1, 0, 2, 5, 3]
>   输出：2 或 3 

```java
 public int findRepeatNumber(int[] nums) {
        //hash表
        int[] arr = new int[nums.length];
        for(int i = 0; i < nums.length; i++){
            arr[nums[i]]++;
            if(arr[nums[i]] > 1) return nums[i];
        }
        return -1;
    }
```

# [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

>   给定一个二叉树，找出其最大深度。
>
>   二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
>
>   **说明:** 叶子节点是指没有子节点的节点。

```java
   public int maxDepth(TreeNode root) {
        if(root == null) return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
```

# [剑指 Offer 24 - 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

>   递归也要熟练的写哦

```java
 public ListNode reverseList(ListNode head) {
        ListNode pre = head;
        ListNode next = null;
        while (pre != null) {
            ListNode temp = pre.next;
            pre.next = next;
            next = pre;
            pre = temp;
        }
        return next;
    }
```

```java
 public ListNode reverseList(ListNode head) {
   			//base case
        if(head == null || head.next == null) {
            return head;
        }
   			//递归
        ListNode node = reverseList(head.next);
   			//关键的一步
        head.next.next = head;
   			//置空
        head.next = null;
        return node;
    }
```



# [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

>   给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
>
>   设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
>
>   注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>    
>
>   示例 1:
>
>   输入: [7,1,5,3,6,4]
>   输出: 7
>   解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
>        随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
>   示例 2:
>
>   输入: [1,2,3,4,5]
>   输出: 4
>   解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
>        注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
>        因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
>   示例 3:
>
>   输入: [7,6,4,3,1]
>   输出: 0
>   解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
>
>
>   提示：
>
>   1 <= prices.length <= 3 * 10 ^ 4
>   0 <= prices[i] <= 10 ^ 4

```java
  public int maxProfit(int[] prices) {
        int max = 0;
        for(int i = 1;i<prices.length;i++){
            if(prices[i] > prices[i-1]){
                max += prices[i] - prices[i-1];
            }
        }
        return max;
    }
```

# [76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

>   给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
>
>   注意：如果 s 中存在这样的子串，我们保证它是唯一的答案。
>
>    
>
>   示例 1：
>
>   输入：s = "ADOBECODEBANC", t = "ABC"
>   输出："BANC"
>   示例 2：
>
>   输入：s = "a", t = "a"
>   输出："a"
>
>
>   提示：
>
>   1 <= s.length, t.length <= 105
>   s 和 t 由英文字母组成

```java
 public String minWindow(String s, String t) {
        int sLen = s.length();
        int tLen = t.length();

        if (sLen == 0 || tLen == 0 || sLen < tLen) {
            return "";
        }

        char[] charArrayS = s.toCharArray();
        char[] charArrayT = t.toCharArray();

        int[] tFreq = new int[128];
        for (char c : charArrayT) {
            tFreq[c]++;
        }

        // 滑动窗口内部还差多少 T 中的字符，对应字符频数超过不重复计算
        int distance = tLen;
        int minLen = sLen + 1;
        int begin = 0;

        int left = 0;
        int right = 0;
        // [left..right)
        while (right < sLen) {
            char charRight = charArrayS[right];
            if (tFreq[charRight] > 0) {
                distance--;
            }
            tFreq[charRight]--;
            right++;

            // System.out.println(distance + " " + s.substring(left, right));
            while (distance == 0) {
                // System.out.println("左边界收缩 " + distance + " " + s.substring(left, right));
                // System.out.println(tFreq['A'] + "," + tFreq['B'] + "," + tFreq['C']);

                if (right - left < minLen) {
                    minLen = right - left;
                    begin = left;
                }

                char charLeft = charArrayS[left];
                tFreq[charLeft]++;
                if (tFreq[charLeft] > 0) {
                    distance++;
                }
                left++;
            }
        }

        if (minLen == sLen + 1) {
            return "";
        }
        return s.substring(begin, begin + minLen);
    }
```

# [8. 字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

>   请你来实现一个 atoi 函数，使其能将字符串转换成整数。
>
>   首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。接下来的转化规则如下：
>
>   如果第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字字符组合起来，形成一个有符号整数。
>   假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成一个整数。
>   该字符串在有效的整数部分之后也可能会存在多余的字符，那么这些字符可以被忽略，它们对函数不应该造成影响。
>   注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换，即无法进行有效转换。
>
>   在任何情况下，若函数不能进行有效的转换时，请返回 0 。
>
>   提示：
>
>   本题中的空白字符只包括空格字符 ' ' 。
>   假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−231,  231 − 1]。如果数值超过这个范围，请返回  INT_MAX (231 − 1) 或 INT_MIN (−231) 。
>
>
>   示例 1:
>
>   输入: "42"
>   输出: 42
>   示例 2:
>
>   输入: "   -42"
>   输出: -42
>   解释: 第一个非空白字符为 '-', 它是一个负号。
>       我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
>   示例 3:
>
>   输入: "4193 with words"
>   输出: 4193
>   解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
>   示例 4:
>
>   输入: "words and 987"
>   输出: 0
>   解释: 第一个非空字符是 'w', 但它不是数字或正、负号。
>       因此无法执行有效的转换。
>   示例 5:
>
>   输入: "-91283472332"
>   输出: -2147483648
>   解释: 数字 "-91283472332" 超过 32 位有符号整数范围。 
>       因此返回 INT_MIN (−231) 。

### 状态机

字符串处理的题目往往涉及复杂的流程以及条件情况，如果直接上手写程序，一不小心就会写出极其臃肿的代码。

因此，为了有条理地分析每个输入字符的处理方法，我们可以使用自动机这个概念：

我们的程序在每个时刻有一个状态 s，每次从序列中输入一个字符 c，并根据字符 c 转移到下一个状态 s'。这样，我们只需要建立一个覆盖所有情况的从 s 与 c 映射到 s' 的表格即可解决题目中的问题。

本题可以建立如下图所示的自动机：

![fig1](https://assets.leetcode-cn.com/solution-static/8_fig1.PNG)

```java
 class Automaton {
        //初始状态
        final String START = "start";
        //有符号
        final String SIGNED = "signed";
        //数字
        final String NUM = "number";
        //结束
        final String END = "end";

        String state = START;
        Map<String, String[]> map;
        int sign = 1;
        long ans = 0;

        /**
         * 什么意思呢?顿悟了已经哈哈哈舒服
         */
        public Automaton() {
            map = new HashMap<>();
            //牛逼啊,getCol的返回值决定了这个string的顺序
            //
            map.put(START, new String[]{START, SIGNED, NUM, END});
            map.put(SIGNED, new String[]{END, END, NUM, END});
            map.put(NUM, new String[]{END, END, NUM, END});
            map.put(END, new String[]{END, END, END, END});
        }

        public int getCol(char c) {
            if (c == ' ') return 0;
            if (c == '+' || c == '-') return 1;
            if (c >= '0' && c <= '9') return 2;
            return 3;
        }

        /**
         * @param c
         */
        public void get(char c) {
            //
            state = map.get(state)[getCol(c)];
            if (state.equals(NUM)) {
                ans = ans * 10 + c - '0';
                if (sign == 1) {
                    ans = Math.min(ans, Integer.MAX_VALUE);
                } else {
                    // -(long)Integer.MIN_VALUE，这个操作有点东西，不然越界
                    ans = Math.min(ans, -(long) Integer.MIN_VALUE);
                }
            } else if (state.equals(SIGNED))
                sign = c == '+' ? 1 : -1;
        }
    }

    public int myAtoiRaw(String str) {
        Automaton automaton = new Automaton();
        char[] c = str.toCharArray();
        for (char ch : c) {
            automaton.get(ch);
        }
        return automaton.sign * ((int) automaton.ans);
    }
```



### 朴素法

```java
   public int myAtoi(String str) {
        //先trim一下,去掉空格
        str = str.trim();
        //如果长度是0返回0
        if (str.length() == 0) return 0;
        //如果开头不是数字或者符号,返回0
        if (!Character.isDigit(str.charAt(0))
                && str.charAt(0) != '-' && str.charAt(0) != '+')
            return 0;
        //防止溢出
        long ans = 0L;
        //负数
        boolean neg = str.charAt(0) == '-';
        //如果不是数字,
        int i = !Character.isDigit(str.charAt(0)) ? 1 : 0;
        //如果只有一个符号,那么肯定是+或者- 且必须是数字,数字中有任何一个符号就直接返回
        while (i < str.length() && Character.isDigit(str.charAt(i))) {
            //获取第一个数字,每次都要乘10
            ans = ans * 10 + (str.charAt(i++) - '0');
            //溢出判断
            if (!neg && ans > Integer.MAX_VALUE) {
                ans = Integer.MAX_VALUE;
                break;
            }
            //没有溢出的话,负数也会溢出,哈哈,所以溢出也要考虑两种情况
            if (neg && ans > 1L + Integer.MAX_VALUE) {
                ans = 1L + Integer.MAX_VALUE;
                break;
            }
            //其他情况继续循环
        }
        //正负判断
        return neg ? (int) -ans : (int) ans;
    }
```

# [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

>   给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。
>
>   求在该柱状图中，能够勾勒出来的矩形的最大面积。
>
>   ![image-20201124133321507](/Users/lumac/Library/Application Support/typora-user-images/image-20201124133321507.png)

```java
  public int largestRectangleArea(int[] heights) {
        Stack < Integer > stack = new Stack < > ();
        stack.push(-1);
        int maxarea = 0;
        for (int i = 0; i < heights.length; ++i) {
            while (stack.peek() != -1 && heights[stack.peek()] >= heights[i])
                maxarea = Math.max(maxarea, heights[stack.pop()] * (i - stack.peek() - 1));
            stack.push(i);
        }
        while (stack.peek() != -1)
            maxarea = Math.max(maxarea, heights[stack.pop()] * (heights.length - stack.peek() -1));
        return maxarea;
    }
```

```java
public int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0)return 0;

        int[] lessFromRight = new int[heights.length];
        int[] lessFromLeft = new int[heights.length];
        lessFromRight[heights.length - 1] = heights.length;
        lessFromLeft[0] = -1;

        //找左边界
        for(int i = 1;i < heights.length ;i++){
            int p = i - 1;
            while(p >= 0 && heights[p] >= heights[i]){
                p = lessFromLeft[p];
            }
            lessFromLeft[i] = p;
        }
        //找右边界
        for(int i = heights.length - 2;i >=0;i--){
            int p = i + 1;
            while(p < heights.length && heights[p] >= heights[i]){
                p = lessFromRight[p];
            }
            lessFromRight[i] = p;
        }
        //计算面积
        int maxArea = 0;
        for(int i = 0;i < heights.length;i++){
            maxArea = Math.max(maxArea,heights[i] * (lessFromRight[i] - lessFromLeft[i] -1));
        }
        return maxArea;
    }
```



# [128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

>   给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
>
>    
>
>   进阶：你可以设计并实现时间复杂度为 O(n) 的解决方案吗？
>
>    
>
>   示例 1：
>
>   输入：nums = [100,4,200,1,3,2]
>   输出：4
>   解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
>   示例 2：
>
>   输入：nums = [0,3,7,2,5,8,4,6,0,1]
>   输出：9
>
>
>   提示：
>
>   0 <= nums.length <= 104
>   -109 <= nums[i] <= 109

```java
  public int longestConsecutive(int[] nums) {
        Set<Integer> numsSet = new HashSet<>();
        for (Integer num : nums) {
            numsSet.add(num);
        }
        int longest = 0;
        for (Integer num : nums) {
            if (numsSet.remove(num)) {
                // 向当前元素的左边搜索,eg: 当前为100, 搜索：99，98，97,...
                int currentLongest = 1;
                int current = num;
                while (numsSet.remove(current - 1)) current--;
                currentLongest += (num - current);
		// 向当前元素的右边搜索,eg: 当前为100, 搜索：101，102，103,...
                current = num;
                while(numsSet.remove(current + 1)) current++;
                currentLongest += (current - num);
        	// 搜索完后更新longest.
                longest = Math.max(longest, currentLongest);
            }
        }
        return longest;
    }
```

# [347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

>   给定一个非空的整数数组，返回其中出现频率前 k 高的元素。
>
>    
>
>   示例 1:
>
>   输入: nums = [1,1,1,2,2,3], k = 2
>   输出: [1,2]
>   示例 2:
>
>   输入: nums = [1], k = 1
>   输出: [1]
>
>
>   提示：
>
>   你可以假设给定的 k 总是合理的，且 1 ≤ k ≤ 数组中不相同的元素的个数。
>   你的算法的时间复杂度必须优于 O(n log n) , n 是数组的大小。
>   题目数据保证答案唯一，换句话说，数组中前 k 个高频元素的集合是唯一的。
>   你可以按任意顺序返回答案。

```java
   public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> occurrences = new HashMap<Integer, Integer>();
        for (int num : nums) {
            occurrences.put(num, occurrences.getOrDefault(num, 0) + 1);
        }

        // int[] 的第一个元素代表数组的值，第二个元素代表了该值出现的次数
        PriorityQueue<int[]> queue = new PriorityQueue<int[]>(new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                return m[1] - n[1];
            }
        });
        for (Map.Entry<Integer, Integer> entry : occurrences.entrySet()) {
            int num = entry.getKey(), count = entry.getValue();
            if (queue.size() == k) {
                if (queue.peek()[1] < count) {
                    queue.poll();
                    queue.offer(new int[]{num, count});
                }
            } else {
                queue.offer(new int[]{num, count});
            }
        }
        int[] ret = new int[k];
        for (int i = 0; i < k; ++i) {
            ret[i] = queue.poll()[0];
        }
        return ret;
    }
```

# [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

>   给定一个二叉树，判断其是否是一个有效的二叉搜索树。
>
>   假设一个二叉搜索树具有如下特征：
>
>   节点的左子树只包含小于当前节点的数。
>   节点的右子树只包含大于当前节点的数。
>   所有左子树和右子树自身必须也是二叉搜索树。
>   示例 1:
>
>   输入:
>       2
>      / \
>     1   3
>   输出: true
>   示例 2:
>
>   输入:
>       5
>      / \
>     1   4
>        / \
>       3   6
>   输出: false
>   解释: 输入为: [5,1,4,null,null,3,6]。
>        根节点的值为 5 ，但是其右子节点值为 4 。

```java
    public boolean isValidBST(TreeNode root) {
        return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    public boolean validate(TreeNode node, long min, long max) {
        if (node == null) {
            return true;
        }
        if (node.val <= min || node.val >= max) {
            return false;
        }
        return  validate(node.left, min, node.val) && validate(node.right, node.val, max);
    }
```

基于方法一中提及的性质，我们可以进一步知道二叉搜索树「中序遍历」得到的值构成的序列一定是升序的，这启示我们在中序遍历的时候实时检查当前节点的值是否大于前一个中序遍历到的节点的值即可。如果均大于说明这个序列是升序的，整棵树是二叉搜索树，否则不是，下面的代码我们使用栈来模拟中序遍历的过程。

```java
public boolean isValidBST(TreeNode root) {
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        double inorder = -Double.MAX_VALUE;

        while (!stack.isEmpty() || root != null) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
              // 如果中序遍历得到的节点的值小于等于前一个 inorder，说明不是二叉搜索树
            if (root.val <= inorder) {
                return false;
            }
            inorder = root.val;
            root = root.right;
        }
        return true;
    }
```



# [445. 两数相加 II](https://leetcode-cn.com/problems/add-two-numbers-ii/)

>   
>   给你两个 **非空** 链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。
>
>   你可以假设除了数字 0 之外，这两个数字都不会以零开头。
>
>    
>
>   **进阶：**
>
>   如果输入链表不能修改该如何处理？换句话说，你不能对列表中的节点进行翻转。
>
>    
>
>   **示例：**
>
>   ```sql
>   输入：(7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
>   输出：7 -> 8 -> 0 -> 7
>   ```

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
         if (l1 == null) {
            return l2;
        }    
        if (l2 == null) {
            return l1;
        }
        
        Stack<Integer> stack1 = new Stack<>();
        Stack<Integer> stack2 = new Stack<>();
        while (l1 != null) {
            stack1.push(l1.val);
            l1 = l1.next;
        }
        while (l2 != null) {
            stack2.push(l2.val);
            l2 = l2.next;
        }
        
        int carry = 0;
        ListNode head = null;
        while (!stack1.isEmpty() || !stack2.isEmpty() || carry > 0) {
            int sum = carry;
            sum += stack1.isEmpty()? 0: stack1.pop();
            sum += stack2.isEmpty()? 0: stack2.pop();
            ListNode node = new ListNode(sum % 10);
            node.next = head;
            head = node;
            carry = sum / 10;
        }
        return head;
    }
```

# [43. 字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)

> 
> 给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。
>
> **示例 1:**
>
> ```
> 输入: num1 = "2", num2 = "3"
> 输出: "6"
> ```
>
> **示例 2:**
>
> ```
> 输入: num1 = "123", num2 = "456"
> 输出: "56088"
> ```
>
> **说明：**
>
> 1. `num1` 和 `num2` 的长度小于110。
> 2. `num1` 和 `num2` 只包含数字 `0-9`。
> 3. `num1` 和 `num2` 均不以零开头，除非是数字 0 本身。
> 4. **不能使用任何标准库的大数类型（比如 BigInteger）**或**直接将输入转换为整数来处理**。

## 方法1

```java

        public static String multiply(String num1, String num2) {
            if (num1.equals("0") || num2.equals("0")) {
                return "0";
            }
            String ans = "0";
            int m = num1.length(), n = num2.length();
            for (int i = n - 1; i >= 0; i--) {
                StringBuffer curr = new StringBuffer();
                int carry = 0;
                //补0 注意是j > i
                for (int j = n - 1; j > i; j--) {
                    curr.append(0);
                }
                //取第一位数
                int y = num2.charAt(i) - '0';
                //开始一位一位的乘
                for (int j = m - 1; j >= 0; j--) {
                    int x = num1.charAt(j) - '0';
                    int product = x * y + carry;
                    curr.append(product % 10);
                    carry = product / 10;
                }
                //最后一位
                if (carry != 0) {
                    curr.append(carry % 10);
                }
                ans = addStrings(ans, curr.reverse().toString());
            }
            return ans;
        }
    }

    public static String addStrings(String num1, String num2) {
        int i = num1.length() - 1, j = num2.length() - 1, add = 0;
        StringBuffer ans = new StringBuffer();
        while (i >= 0 || j >= 0 || add != 0) {
            int x = i >= 0 ? num1.charAt(i) - '0' : 0;
            int y = j >= 0 ? num2.charAt(j) - '0' : 0;
            int result = x + y + add;
            ans.append(result % 10);
            add = result / 10;
            i--;
            j--;
        }
        ans.reverse();
        return ans.toString();
    }
```

时间复杂度:$$O(m*n+n^2)$$

空间复杂度$$O(m+n)$$

## 方法二

两个字符串的长度为n,m且他们均不为0,他们乘积的长度为m+n-1或者m+n.

巧妙的数组法,牛逼

```java
public String multiply(String num1, String num2) {
  			//基本的判断
        if (num1.equals("0") || num2.equals("0")) {
            return "0";
        }
  			//保存两个长度
        int m = num1.length(), n = num2.length();
  			//保存每次结果的数组
        int[] ansArr = new int[m + n];
  			//
        for (int i = m - 1; i >= 0; i--) {
            int x = num1.charAt(i) - '0';
            for (int j = n - 1; j >= 0; j--) {
                int y = num2.charAt(j) - '0';
                ansArr[i + j + 1] += x * y;
            }
        }
  			//
        for (int i = m + n - 1; i > 0; i--) {
            ansArr[i - 1] += ansArr[i] / 10;
            ansArr[i] %= 10;
        }
        int index = ansArr[0] == 0 ? 1 : 0;
        StringBuffer ans = new StringBuffer();
        while (index < m + n) {
            ans.append(ansArr[index]);
            index++;
        }
        return ans.toString();
    }
```

时间复杂度:$$O(m*n)$$

空间复杂度$$O(m+n)$$

# [6. Z 字形变换](https://leetcode-cn.com/problems/zigzag-conversion/)

>   将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。
>
>   比如输入字符串为 "LEETCODEISHIRING" 行数为 3 时，排列如下：
>
>   L   C   I   R
>   E T O E S I I G
>   E   D   H   N
>   之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："LCIRETOESIIGEDHN"。
>
>   请你实现这个将字符串进行指定行数变换的函数：
>
>   string convert(string s, int numRows);
>   示例 1:
>
>   输入: s = "LEETCODEISHIRING", numRows = 3
>   输出: "LCIRETOESIIGEDHN"
>   示例 2:
>
>   输入: s = "LEETCODEISHIRING", numRows = 4
>   输出: "LDREOEIIECIHNTSG"
>   解释:
>
>   L     D     R
>   E   O E   I I
>   E C   I H   N
>   T     S     G

```java
 public String convert(String s, int numRows) {
        if (s.length() <= numRows || numRows == 1) {
            return s;
        }
        int v = (numRows - 1) * 2;
        int len = s.length();
        char[] buf = new char[len];
        int index = 0;
        for (int i = 0; i < numRows; i++) {
            for (int j = i; j < len && index < len; j += v) {
                buf[index++] = s.charAt(j);
                // 中间行另外处理
                if (i > 0 && i < numRows -1) {
                    int t = j + v - 2 * i;
                    if (t < len) {
                        buf[index++] = s.charAt(t);
                    }
                }
            }
        }
        return new String(buf);
    }
```

#[69. x 的平方根](https://leetcode-cn.com/problems/sqrtx/)

>   
>   实现 `int sqrt(int x)` 函数。
>
>   计算并返回 *x* 的平方根，其中 *x* 是非负整数。
>
>   由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。
>
>   **示例 1:**
>
>   ```
>   输入: 4
>   输出: 2
>   ```
>
>   **示例 2:**
>
>   ```sql
>   输入: 8
>   输出: 2
>   说明: 8 的平方根是 2.82842..., 
>        由于返回类型是整数，小数部分将被舍去。
>   ```

```java
 public int mySqrt(int x) {
        if(x == 0) {
            return 0;
        }
        if(x < 4) {
            return 1;
        }
        int left = 2;
        int right = x / 2;
        
        while(left + 1 < right) {
            int num = (left + right) / 2;
            long sq = (long)num * num;
            if(x < sq) {
                right = num;
            }
            if(x > sq) {
                left = num;
            }
            if(x == sq) {
                return num;
            }
        }
        return left;
    }
```

# [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

>   给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
>   **说明：**每次只能向下或者向右移动一步。
>
>   ![img](https://assets.leetcode.com/uploads/2020/11/05/minpath.jpg)

```java
    public int minPathSum(int[][] grid) {
           if (grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }
        int rows = grid.length, columns = grid[0].length;
        int[][] dp = new int[rows][columns];
        dp[0][0] = grid[0][0];
        for (int i = 1; i < rows; i++) {
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }
        for (int j = 1; j < columns; j++) {
            dp[0][j] = dp[0][j - 1] + grid[0][j];
        }
        for (int i = 1; i < rows; i++) {
            for (int j = 1; j < columns; j++) {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }
        return dp[rows - 1][columns - 1];
    }
```

#[45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

>   给定一个非负整数数组，你最初位于数组的第一个位置。
>
>   数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
>   你的目标是使用最少的跳跃次数到达数组的最后一个位置。
>
>   示例:
>
>   输入: [2,3,1,1,4]
>   输出: 2
>   解释: 跳到最后一个位置的最小跳跃数是 2。
>        从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
>   说明:
>
>   假设你总是可以到达数组的最后一个位置。
>

```java
  public int jump(int[] nums) {
        int end = 0;
        int maxPosition = 0; 
        int steps = 0;
        for(int i = 0; i < nums.length - 1; i++){
        //找能跳的最远的
            maxPosition = Math.max(maxPosition, nums[i] + i); 
            if( i == end){ //遇到边界，就更新边界，并且步数加一
                end = maxPosition;
                steps++;
            }
    }
        return steps;
    }
```

# [84 - 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

>  给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。
>
> 求在该柱状图中，能够勾勒出来的矩形的最大面积。
>
> ![image-20201124145413819](/Users/lumac/Library/Application Support/typora-user-images/image-20201124145413819.png)

```java
  public int largestRectangleArea(int[] heights) {
        Stack < Integer > stack = new Stack < > ();
        stack.push(-1);
        int maxarea = 0;
        for (int i = 0; i < heights.length; ++i) {
            while (stack.peek() != -1 && heights[stack.peek()] >= heights[i])
                maxarea = Math.max(maxarea, heights[stack.pop()] * (i - stack.peek() - 1));
            stack.push(i);
        }
        while (stack.peek() != -1)
            maxarea = Math.max(maxarea, heights[stack.pop()] * (heights.length - stack.peek() -1));
        return maxarea;
    }
```



# [85. 最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

   [84 - 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

>   给定一个仅包含 `0` 和 `1` 、大小为 `rows x cols` 的二维二进制矩阵，找出只包含 `1` 的最大矩形，并返回其面积。
>
>   ![img](https://assets.leetcode.com/uploads/2020/09/14/maximal.jpg)

```java
 public int maximalRectangle(char[][] matrix) {
        if (matrix == null || matrix.length == 0) return 0;
        int n = matrix[0].length;
        int res = 0;
        int[] height = new int[n];
        int[] left = new int[n];
        int[] right = new int[n];
        Arrays.fill(right, n);

        for (char[] chars : matrix) {
            int curLeft = 0, curRight = n;

            for (int j = 0; j < n; j++) {
                if (chars[j] == '1') height[j]++;
                else height[j] = 0;
            }

            for (int j = 0; j < n; j++) {
                if (chars[j] == '1') {
                    left[j] = Math.max(curLeft, left[j]);
                } else {
                    left[j] = 0;
                    curLeft = j + 1;
                }
            }

            for (int j = n - 1; j >= 0; j--) {
                if (chars[j] == '1') {
                    right[j] = Math.min(curRight, right[j]);
                } else {
                    right[j] = n;
                    curRight = j;
                }
            }

            for (int j = 0; j < n; j++) {
                res = Math.max(res, height[j] * (right[j] - left[j]));
            }
        }
        return res;
    }
```

#[695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

>   给定一个包含了一些 0 和 1 的非空二维数组 grid 。
>
>   一个 岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在水平或者竖直方向上相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。
>
>   找到给定的二维数组中最大的岛屿面积。(如果没有岛屿，则返回面积为 0 。)
>
>    ```sql
>   示例 1:
>   
>   [[0,0,1,0,0,0,0,1,0,0,0,0,0],
>    [0,0,0,0,0,0,0,1,1,1,0,0,0],
>    [0,1,1,0,1,0,0,0,0,0,0,0,0],
>    [0,1,0,0,1,1,0,0,1,0,1,0,0],
>    [0,1,0,0,1,1,0,0,1,1,1,0,0],
>    [0,0,0,0,0,0,0,0,0,0,1,0,0],
>    [0,0,0,0,0,0,0,1,1,1,0,0,0],
>    [0,0,0,0,0,0,0,1,1,0,0,0,0]]
>   对于上面这个给定矩阵应返回 6。注意答案不应该是 11 ，因为岛屿只能包含水平或垂直的四个方向的 1 。
>   示例 2:
>   
>   [[0,0,0,0,0,0,0,0]]
>   对于上面这个给定的矩阵, 返回 0。
>    ```
>
>   

```java
 public int maxAreaOfIsland(int[][] grid) {
        int max = 0;
        for(int i = 0;i< grid.length;i++){
            for(int j = 0;j< grid[0].length;j++){
                if (grid[i][j] == 1) {
                    max = Math.max(max,dfs(grid,i,j));
                }
              
            }
        }
        return max;    
    }
    private int dfs(int[][]grid,int i,int j){
        if(i < 0 
        || i >= grid.length
        || j < 0
        || j >= grid[0].length
        || grid[i][j] == 0
        ){
            return 0;
        }
            int count = 1;
            grid[i][j] = 0;
            count += dfs(grid,i-1,j);
            count += dfs(grid,i+1,j);
            count += dfs(grid,i,j-1);
            count += dfs(grid,i,j+1);
            return count;
    }
```

#[26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

>   给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
>
>   不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。
>
>    
>
>   示例 1:
>
>   给定数组 nums = [1,1,2], 
>
>   函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 
>
>   你不需要考虑数组中超出新长度后面的元素。
>   示例 2:
>
>   给定 nums = [0,0,1,1,1,2,2,3,3,4],
>
>   函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。
>
>   你不需要考虑数组中超出新长度后面的元素。
>

```java
  public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;  
        //慢指针,如果元素相同就一直停留,更新的时候加1,更新
        int number = 0;
        for (int i=0; i < nums.length; i++) {  
            if (nums[i] != nums[number] ) {  
                number++;  
                nums[number] = nums[i];  
            }  
        }  
        number+=1; //标记+1即为数字个数  
        return number;  
    }
```

#[240. 搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

>   
>   编写一个高效的算法来搜索 *m* x *n* 矩阵 matrix 中的一个目标值 target。该矩阵具有以下特性：
>
>   - 每行的元素从左到右升序排列。
>   - 每列的元素从上到下升序排列。
>
>   **示例:**
>
>   现有矩阵 matrix 如下：
>
>   ```
>   [
>     [1,   4,  7, 11, 15],
>     [2,   5,  8, 12, 19],
>     [3,   6,  9, 16, 22],
>     [10, 13, 14, 17, 24],
>     [18, 21, 23, 26, 30]
>   ]
>   ```
>
>   给定 target = `5`，返回 `true`。
>
>   给定 target = `20`，返回 `false`。

- 选左上角，往右走和往下走都增大，不能选
- 选右下角，往上走和往左走都减小，不能选
- 选左下角，往右走增大，往上走减小，可选
- 选右上角，往下走增大，往左走减小，可选

```java
   public boolean searchMatrix(int[][] matrix, int target) {
        // start our "pointer" in the bottom-left
        int row = matrix.length - 1;
        int col = 0;

        while (row >= 0 && col < matrix[0].length) {
            if (matrix[row][col] > target) {
                row--;
            } else if (matrix[row][col] < target) {
                col++;
            } else { // found it
                return true;
            }
        }

        return false;
    }
```

