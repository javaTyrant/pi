[toc]

# [50. Pow(x, n)](https://leetcode-cn.com/problems/powx-n/)

>   实现 pow(x, n) ，即计算 x 的 n 次幂函数。
>
>   示例 1:
>
>   输入: 2.00000, 10
>   输出: 1024.00000
>   示例 2:
>
>   输入: 2.10000, 3
>   输出: 9.26100
>   示例 3:
>
>   输入: 2.00000, -2
>   输出: 0.25000
>   解释: 2-2 = 1/22 = 1/4 = 0.25
>   说明:
>
>   -100.0 < x < 100.0
>   n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 。

「快速幂算法」的本质是分治算法。

> *x*→*x*2→*x*4→*x*8→*x*16→*x*32→*x*64
>
> *x*→*x*2→*x*4→*x*9→*x*19→*x*38→*x*77

```java
    public double myPow(double x, int n) {
        double res = 1.0;
        for(int i = n; i != 0; i /= 2){
            //奇数 = 乘一次
            if((i & 1) != 0 ){
                res *= x;
            }
            x *= x;
        }
        return  n < 0 ? 1 / res : res;
    }
```

```java
    double quickMul(double x, long N) {
        double ans = 1.0;
        // 贡献的初始值为 x
        double x_contribute = x;
        // 在对 N 进行二进制拆分的同时计算答案
        while (N > 0) {
            if (N % 2 == 1) {
                // 如果 N 二进制表示的最低位为 1，那么需要计入贡献
                ans *= x_contribute;
            }
            // 将贡献不断地平方
            x_contribute *= x_contribute;
            // 舍弃 N 二进制表示的最低位，这样我们每次只要判断最低位即可
            N /= 2;
        }
        return ans;
    }

    public double myPow(double x, int n) {
        long N = n;
        return N >= 0 ? quickMul(x, N) : 1.0 / quickMul(x, -N);
    }
}
```



#[13. 罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/)

>   罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。
>
>   字符          数值
>   I             1
>   V             5
>   X             10
>   L             50
>   C             100
>   D             500
>   M             1000
>   例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。
>
>   通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：
>
>   I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
>   X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
>   C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。
>   给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。
>
>   示例 1:
>
>   输入: "III"
>   输出: 3
>   示例 2:
>
>   输入: "IV"
>   输出: 4
>   示例 3:
>
>   输入: "IX"
>   输出: 9
>   示例 4:
>
>   输入: "LVIII"
>   输出: 58
>   解释: L = 50, V= 5, III = 3.
>   示例 5:
>
>   输入: "MCMXCIV"
>   输出: 1994
>   解释: M = 1000, CM = 900, XC = 90, IV = 4.

```java
  //很聪明的解法
    public int romanToInt(String s) {
        Map<Character, Integer> map = new HashMap<>();
        map.put('I', 1);
        map.put('V', 5);
        map.put('X', 10);
        map.put('L', 50);
        map.put('C', 100);
        map.put('D', 500);
        map.put('M', 1000);
        int res = 0;
        for(int i = 0;i < s.length();i++){
          	//先把值取出来
            int cur = map.get(s.charAt(i));
            //不越界,取下一个值,越界则取0
            int nextVal = i < s.length() - 1 ? map.get(s.charAt(i + 1)) : 0;
            //当前值小于下一个值IV减,否则+
            res += cur < nextVal ? - cur : cur;
        }
        return res;
    }
```

# [51. N 皇后](https://leetcode-cn.com/problems/n-queens/)

时间复杂度：O(N!)*O*(*N*!)，其中 N*N* 是皇后数量。

空间复杂度：O(N)*O*(*N*)

>   *n* 皇后问题研究的是如何将 *n* 个皇后放置在 *n*×*n* 的棋盘上，并且使皇后彼此之间不能相互攻击。
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/8-queens.png)

```java
		//N皇后
    List<List<String>> list;//保存结果
		//入口
    public List<List<String>> solveNQueens(int n) {
        list = new ArrayList<>();
        int[] x = new int[n];//保存结果
        queens(x, n, 0);
        return list;
    }

    void queens(int[] x, int n, int row) {
        for (int i = 0; i < n; i++) {
            if (check(x, n, row, i)) {//判断合法
                x[row] = i;//将皇后放在第row行，第i列
                if (row == n - 1) {//如果是最后一行，则输出结果
                    addList(x, n);
                    x[row] = 0;//回溯，寻找下一个结果
                    return;
                }
                queens(x, n, row + 1);//寻找下一行
                x[row] = 0;//回溯
            }
        }
    }

    /**
     * 将每一组的结果添加list
     *
     * @param x 数组解
     * @param n 棋盘长宽
     */
    private void addList(int[] x, int n) {
        //添加结果
        String[][] sArr = new String[n][n];
        List<String> al = new ArrayList<String>();
        for (int i = 0; i < n; i++) {
            Arrays.fill(sArr[i], ".");//先填充.
            sArr[i][x[i]] = "Q";//特定位置放置Q
            String s = "";
            for (String str : sArr[i]) {
                s += str;//加在一起
            }
            al.add(s);//添加一行
        }
        list.add(al);//添加一组解
    }

    boolean check(int[] x, int n, int row, int col) {
        for (int i = 0; i < row; i++) {
            //因为行不相等，判断列是否相等，斜线上是否相等
            if (x[i] == col || x[i] + i == col + row || x[i] - i == col - row)
                return false;
        }
        return true;
    }
```

# [155. 最小栈](https://leetcode-cn.com/problems/min-stack/)

>   设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
>
>   push(x) —— 将元素 x 推入栈中。
>   pop() —— 删除栈顶的元素。
>   top() —— 获取栈顶元素。
>   getMin() —— 检索栈中的最小元素。

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
    
    public int getMin() {
       return node.min;
    }
}
```

# [287. 寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)

>   
>   给定一个包含 *n* + 1 个整数的数组 *nums*，其数字都在 1 到 *n* 之间（包括 1 和 *n*），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。
>
>   **示例 1:**
>
>   ```
>   输入: [1,3,4,2,2]
>   输出: 2
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: [3,1,3,4,2]
>   输出: 3
>   ```
>
>   **说明：**
>
>   1. **不能**更改原数组（假设数组是只读的）。
>   2. 只能使用额外的 *O*(1) 的空间。
>   3. 时间复杂度小于 *O*(*n*2) 。
>   4. 数组中只有一个重复的数字，但它可能不止重复出现一次。

```java
  public int findDuplicate(int[] nums) {
        int slow = 0;
        int fast = 0;
        do{
           slow = nums[slow];
           fast = nums[nums[fast]]; 
        }while(slow != fast);
        slow = 0;
        while(slow != fast){
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    }
 public int findDuplicate(int[] nums) {
        int n = nums.length, ans = 0;
        int bit_max = 31;
        while (((n - 1) >> bit_max) == 0) {
            bit_max -= 1;
        }
        for (int bit = 0; bit <= bit_max; ++bit) {
            int x = 0, y = 0;
            for (int i = 0; i < n; ++i) {
                if ((nums[i] & (1 << bit)) != 0) {
                    x += 1;
                }
                if (i >= 1 && ((i & (1 << bit)) != 0)) {
                    y += 1;
                }
            }
            if (x > y) {
                ans |= 1 << bit;
            }
        }
        return ans;
    }
```

# [91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

>   一条包含字母 A-Z 的消息通过以下方式进行了编码：
>
>   'A' -> 1
>   'B' -> 2
>   ...
>   'Z' -> 26
>   给定一个只包含数字的非空字符串，请计算解码方法的总数。
>
>   题目数据保证答案肯定是一个 32 位的整数。
>
>   示例 1：
>
>   输入：s = "12"
>   输出：2
>   解释：它可以解码为 "AB"（1 2）或者 "L"（12）。
>   示例 2：
>
>   输入：s = "226"
>   输出：3
>   解释：它可以解码为 "BZ" (2 26), "VF" (22 6), 或者 "BBF" (2 2 6) 。
>   示例 3：
>
>   输入：s = "0"
>   输出：0
>   示例 4：
>
>   输入：s = "1"
>   输出：1
>   示例 5：
>
>   输入：s = "2"
>   输出：1

```java
public int numDecodings(String s) {
     
        //dp[i]用于记录字符串至第i-1位前的解码方法的总数
        int[] dp = new int[s.length() + 1];
        dp[0] = 1;

        for(int i = 1; i < s.length() + 1; i++){
            //当前数字不为0时，dp[i] += dp[i - 1]表示当前组合数量包括前一位数字前的组合总数
            char c = s.charAt(i - 1);
            if(c != '0'){
                dp[i] += dp[i - 1];
            }
            //当前数字与前一位组合的数字处于10-26时，dp[i] += dp[i - 2]表示当前组合数量包括前两位数字前的组合总数
            if(i > 1){
                int num = (s.charAt(i - 2) - '0') * 10 + (c - '0');
                //提前剪枝：连续两个0没有对应解码方式
                if(num == 0){
                    return 0;
                }
                if(num > 9 && num < 27){
                    dp[i] += dp[i - 2];
                }
            }
        }

        return s.length() == 0 ? 0 : dp[s.length()];
    }
```

# [剑指 Offer 09 - 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

>   用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )
>

```java
class CQueue {
    Stack<Integer> stack1;
    Stack<Integer> stack2;
    public CQueue() {
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }
    
    public void appendTail(int value) {
        stack1.push(value);
    }
    
    public int deleteHead() {
        if(stack2.size()<=0){
            if(stack1.size() <= 0) return -1;
            int size = stack1.size();
            for(int i = 0;i < size;i++){
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
}

```

# [38. 外观数列](https://leetcode-cn.com/problems/count-and-say/)

>   1.     1
>   2.     11
>   3.     21
>   4.     1211
>   5.     111221
>   第一项是数字 1 
>   描述前一项，这个数是 1 即 “ 一 个 1 ”，记作 "11"
>   描述前一项，这个数是 11 即 “ 二 个 1 ” ，记作 "21"
>   描述前一项，这个数是 21 即 “ 一 个 2 + 一 个 1 ” ，记作 "1211"
>   描述前一项，这个数是 1211 即 “ 一 个 1 + 一 个 2 + 二 个 1 ” ，记作 "111221"
>

```java
public String countAndSay(int n) {
        	if(n==1)return "1";    	
    	String s="1";
    	for(int i=0;i<n-1;i++) {
    		StringBuffer sb=new StringBuffer();
    		char t=s.charAt(0);
    		int count=0;
    		int j=0;
    		for(j=0;j<s.length();j++) {
    			if(t!=s.charAt(j)) {
    				sb.append(count);
    				sb.append(t);
    				t=s.charAt(j);
    				count=1;
    			}
    			else {
    				count++;
    			}
    				
    		}
    		sb.append(count);
    		sb.append(s.charAt(s.length()-1));
    		s=sb.toString();
    		
    		
    	}
    	
        return s;
    }
```

# [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

>   给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。
>
>   示例 1:
>
>   输入: 1->1->2
>   输出: 1->2
>   示例 2:
>
>   输入: 1->1->2->3->3
>   输出: 1->2->3

```java
 public ListNode deleteDuplicates(ListNode head) {
        ListNode current = head;
        while (current != null && current.next != null) {
            if (current.next.val == current.val) {
                current.next = current.next.next;
            } else {
                current = current.next;
            }
        }
        return head;
    }
```

# [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)

>   序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。
>
>   请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。
>
>   示例: 
>
>   你可以将以下二叉树：
>
>   ​	1
>
>      / \
>     2   3
>        / \
>       4   5
>
>   序列化为 "[1,2,3,null,null,4,5]"

遍历和反遍历

```java
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

    // Decodes your encoded data to tree.
    public TreeNode deserialize(List<Integer> data) {
        int index[] = {0};
        TreeNode root = build(index, data);
        return root;
    }

    private TreeNode build(int[] index, List<Integer> data) {
        Integer val = data.get(index[0]);
        index[0] = index[0] + 1;
        if (val == null) {
            return null;
        } else {
            TreeNode node = new TreeNode(val);
            node.left = build(index, data);
            node.right = build(index, data);
            return node;
        }
    }
```

#[394. 字符串解码](https://leetcode-cn.com/problems/decode-string/) 

>   给定一个经过编码的字符串，返回它解码后的字符串。
>
>   编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
>
>   你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
>   此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。
>
>    
>
>   示例 1：
>
>   输入：s = "3[a]2[bc]"
>   输出："aaabcbc"
>   示例 2：
>
>   输入：s = "3[a2[c]]"
>   输出："accaccacc"
>   示例 3：
>
>   输入：s = "2[abc]3[cd]ef"
>   输出："abcabccdcdcdef"
>   示例 4：
>
>   输入：s = "abc3[cd]xyz"
>   输出："abccdcdcdxyz"

更加优雅的解决方案

```java
  public static String decodeString_(String s) {
            Stack<Integer> mul = new Stack<>();
            Stack<String> pre = new Stack<>();
            //因为可能是20ab,这种,肯定要把值截取出来
            int num = 0;
            StringBuilder res = new StringBuilder();
            for (char c : s.toCharArray()) {
                if (c <= '9' && c >= '0') {
                    num = num * 10 + c - '0';
                } else if (c == '[') {
                    mul.push(num);
                    num = 0;
                    pre.push(res.toString());
                    res = new StringBuilder();
                } else if (c == ']') {
                    int time = mul.pop();
                    String tmp = pre.pop();
                    StringBuilder sb = new StringBuilder(tmp);
                    while (time-- > 0) {
                        sb.append(res.toString());
                    }
                    res = new StringBuilder(sb.toString());
                } else {
                    res.append(c);
                }
            }
            return res.toString();
        }
```



```java
 int ptr;

    public String decodeString(String s) {
        LinkedList<String> stk = new LinkedList<String>();
        ptr = 0;

        while (ptr < s.length()) {
            char cur = s.charAt(ptr);
            if (Character.isDigit(cur)) {
                // 获取一个数字并进栈
                String digits = getDigits(s);
                stk.addLast(digits);
            } else if (Character.isLetter(cur) || cur == '[') {
                // 获取一个字母并进栈
                stk.addLast(String.valueOf(s.charAt(ptr++))); 
            } else {
                ++ptr;
                LinkedList<String> sub = new LinkedList<String>();
                while (!"[".equals(stk.peekLast())) {
                    sub.addLast(stk.removeLast());
                }
                Collections.reverse(sub);
                // 左括号出栈
                stk.removeLast();
                // 此时栈顶为当前 sub 对应的字符串应该出现的次数
                int repTime = Integer.parseInt(stk.removeLast());
                StringBuffer t = new StringBuffer();
                String o = getString(sub);
                // 构造字符串
                while (repTime-- > 0) {
                    t.append(o);
                }
                // 将构造好的字符串入栈
                stk.addLast(t.toString());
            }
        }

        return getString(stk);
    }

    public String getDigits(String s) {
        StringBuffer ret = new StringBuffer();
        while (Character.isDigit(s.charAt(ptr))) {
            ret.append(s.charAt(ptr++));
        }
        return ret.toString();
    }

    public String getString(LinkedList<String> v) {
        StringBuffer ret = new StringBuffer();
        for (String s : v) {
            ret.append(s);
        }
        return ret.toString();
    }
```

# [面试题 16.25 - LRU缓存](https://leetcode-cn.com/problems/lru-cache-lcci/)

>   设计和构建一个“最近最少使用”缓存，该缓存会删除最近最少使用的项目。缓存应该从键映射到值(允许你插入和检索特定键对应的值)，并在初始化时指定最大容量。当缓存被填满时，它应该删除最近最少使用的项目。
>
>   它应该支持以下操作： 获取数据 get 和 写入数据 put 。
>
>   获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
>   写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

```java

```

# [剑指 Offer 29 - 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

>   输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。
>
>    
>
>   示例 1：
>
>   输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
>   输出：[1,2,3,6,9,8,7,4,5]
>   示例 2：
>
>   输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
>   输出：[1,2,3,4,8,12,11,10,9,5,6,7]

```java
public int[] spiralOrder(int[][] matrix) {
     if(matrix == null || matrix.length == 0) return new int[0];
        int[]result = new int[matrix.length * matrix[0].length];
        int left = 0,right = matrix[0].length - 1,up = 0,down = matrix.length - 1;
        int index = 0;
        while(left <= right && up <= down){
            for(int i = left;i <= right;i++){
                result[index++] = matrix[up][i];
            }
            for(int i = up + 1;i <= down;i++){
                result[index++] = matrix[i][right];
            }
            if(up != down){
                for(int i = right - 1;i >= left;i--){
                    result[index++] = matrix[down][i];
                }
            }
            if(left != right){
                for(int i = down - 1;i > up;i--){
                    result[index++] = matrix[i][left];
                }
            }
            left++;
            right--;
            up++;
            down--;
        }
        return result;
    }
```

# [剑指 Offer 22 - 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

>   输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。
>
>    
>
>   示例：
>
>   给定一个链表: 1->2->3->4->5, 和 k = 2.
>
>   返回链表 4->5.

快慢指针

```java
   public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode frontNode = head, behindNode = head;
        while (frontNode != null && k > 0) {
            frontNode = frontNode.next;
            k--;
        }
        while (frontNode != null) {
            frontNode = frontNode.next;
            behindNode = behindNode.next;
        }
        return behindNode;
    }
```

# [139. 单词拆分](https://leetcode-cn.com/problems/word-break/)

>   给定一个非空字符串 s 和一个包含非空单词的列表 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。
>
>   说明：
>
>   拆分时可以重复使用字典中的单词。
>   你可以假设字典中没有重复的单词。
>
>   示例 1：
>
>   输入: s = "leetcode", wordDict = ["leet", "code"]
>   输出: true
>   解释: 返回 true 因为 "leetcode" 可以被拆分成 "leet code"。
>   示例 2：
>
>   输入: s = "applepenapple", wordDict = ["apple", "pen"]
>   输出: true
>   解释: 返回 true 因为 "applepenapple" 可以被拆分成 "apple pen apple"。
>        注意你可以重复使用字典中的单词。

```java
  public boolean wordBreak(String s, List<String> wordDict) {
        // Set<String> wordDictSet = new HashSet(wordDict);
        // boolean[] dp = new boolean[s.length() + 1];
        // dp[0] = true;
        // for (int i = 1; i <= s.length(); i++) {
        //     for (int j = 0; j < i; j++) {
        //         if (dp[j] && wordDictSet.contains(s.substring(j, i))) {
        //             dp[i] = true;
        //             break;
        //         }
        //     }
        // }
        // return dp[s.length()];
        int len = s.length(), maxw = 0;
        boolean[] dp = new boolean[len + 1]; 
        dp[0] = true;
        Set<String> set = new HashSet();
        for(String str : wordDict){
            set.add(str);
            maxw = Math.max(maxw, str.length());
        }
        for(int i = 1; i < len + 1; i++){
            for(int j = i; j >= 0 && j >= i - maxw; j--){
                if(dp[j] && set.contains(s.substring(j, i))){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[len];
    }
```

# [114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

[144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

>   给定一个二叉树，[原地](https://baike.baidu.com/item/原地算法/8010757)将它展开为一个单链表。
>
>   
>
>   例如，给定二叉树
>
>   ```java
>      1
>     / \
>    2   5
>   / \   \
>   3   4   6
>   ```
>
>   ```java
>   1
>   \
>    2
>     \
>      3
>       \
>        4
>         \
>          5
>           \
>            6
>   ```

递归

```java
 public void flatten(TreeNode root) {
        //退出条件
        if(root == null) return;
        //最终到3
        flatten(root.left);
        flatten(root.right);
        //构建一个tmp,保存root.right
        TreeNode tmp = root.right;
        //left赋值给left
        root.right = root.left;
   			//left赋空
        root.left = null;
   			//当right不为空
        while(root.right != null)
         	root = root.right;
   			//root.right复原
        root.right = tmp;
    }
```

迭代

```java
 public void flatten(TreeNode root) {
        List<TreeNode> list = new ArrayList<TreeNode>();
        Deque<TreeNode> stack = new LinkedList<TreeNode>();
        TreeNode node = root;
        while (node != null || !stack.isEmpty()) {
            while (node != null) {
                list.add(node);
                stack.push(node);
                node = node.left;
            }
            node = stack.pop();
            node = node.right;
        }
        int size = list.size();
        for (int i = 1; i < size; i++) {
            TreeNode prev = list.get(i - 1), curr = list.get(i);
            prev.left = null;
            prev.right = curr;
        }
    }
```



#[167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/) 

>   给定一个已按照升序排列 的有序数组，找到两个数使得它们相加之和等于目标数。
>
>   函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。
>
>   说明:
>
>   返回的下标值（index1 和 index2）不是从零开始的。
>   你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。
>   示例:
>
>   输入: numbers = [2, 7, 11, 15], target = 9
>   输出: [1,2]
>   解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。

```java
   public int[] twoSum(int[] numbers, int target) {
        // int i = 0,j = numbers.length;
        // while(i < j){
        //     int mid = i + (j - i)/2;
        //     if(numbers[i]+numbers[mid] == target){
        //         return new int[]{i+1,mid+1};
        //     }else if(numbers[i]+numbers[mid] < target){
        //             i = mid+1;
        //     }else{
        //             j = mid;
        //     }
        // }
        // return new int[]{-1,-1};
        int l = 0;
        int r = numbers.length - 1;
        int sum = 0;
        while(l < r){
            sum = numbers[l]+numbers[r];
            if(sum > target){
                r--;
            }else if(sum < target){
                l++;
            }else{
                return new int[]{l+1,r+1};
            }
        }
        return new int[]{-1};
    }
```

# [134. 加油站](https://leetcode-cn.com/problems/gas-station/)

>   在一条环路上有 N 个加油站，其中第 i 个加油站有汽油 gas[i] 升。
>
>   你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i+1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。
>
>   如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。
>
>   说明: 
>
>   如果题目有解，该答案即为唯一答案。
>   输入数组均为非空数组，且长度相同。
>   输入数组中的元素均为非负数。
>
>   示例 1:
>
>   输入: 
>   gas  = [1,2,3,4,5]
>   cost = [3,4,5,1,2]
>
>   输出: 3
>
>   解释:
>   从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
>   开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
>   开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
>   开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
>   开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
>   开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
>   因此，3 可为起始索引。
>   示例 2:
>
>   输入: 
>   gas  = [2,3,4]
>   cost = [3,4,3]
>
>   输出: -1
>
>   解释:
>   你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
>   我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油
>   开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油
>   开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油
>   你无法返回 2 号加油站，因为返程需要消耗 4 升汽油，但是你的油箱只有 3 升汽油。
>   因此，无论怎样，你都不可能绕环路行驶一周。

```java
 public int canCompleteCircuit(int[] gas, int[] cost) {
        int cur = 0;
        int total = 0;
        int start = 0;
        for(int i = 0;i<gas.length;i++){
            total += (gas[i] - cost[i]);
             cur += (gas[i] - cost[i]);
             if(cur < 0){
                 cur = 0;
                 start = i+1;
             }
        }
        return total < 0 ? -1: start;
    }
```

# [152. 乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

>   给你一个整数数组 nums ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。
>
>    
>
>   示例 1:
>
>   输入: [2,3,-2,4]
>   输出: 6
>   解释: 子数组 [2,3] 有最大乘积 6。
>   示例 2:
>
>   输入: [-2,0,-1]
>   输出: 0
>   解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。

```java
 public int maxProduct(int[] nums) {
        //一个保存最大的，一个保存最小的。
        int max = Integer.MIN_VALUE, imax = 1, imin = 1;
        for(int i=0; i<nums.length; i++){
            //如果数组的数是负数，那么会导致最大的变最小的，最小的变最大的。因此交换两个的值。
            if(nums[i] < 0){ 
                int tmp = imax; 
                imax = imin;
                imin = tmp;
            } 
            imax = Math.max(imax*nums[i], nums[i]);
            imin = Math.min(imin*nums[i], nums[i]);
            max = Math.max(max, imax);
        }
        return max;
    }
```

# [207. 课程表](https://leetcode-cn.com/problems/course-schedule/)

>   你这个学期必须选修 numCourse 门课程，记为 0 到 numCourse-1 。
>
>   在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们：[0,1]
>
>   给定课程总量以及它们的先决条件，请你判断是否可能完成所有课程的学习？
>
>    
>
>   示例 1:
>
>   输入: 2, [[1,0]] 
>   输出: true
>   解释: 总共有 2 门课程。学习课程 1 之前，你需要完成课程 0。所以这是可能的。
>   示例 2:
>
>   输入: 2, [[1,0],[0,1]]
>   输出: false
>   解释: 总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0；并且学习课程 0 之前，你还应先完成课程 1。这是不可能的。

```java
 public boolean canFinish(int numCourses, int[][] prerequisites) {
        int[] indegrees = new int[numCourses];
        List<List<Integer>> adjacency = new ArrayList<>();
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++)
            adjacency.add(new ArrayList<>());
        // Get the indegree and adjacency of every course.
        for(int[] cp : prerequisites) {
            indegrees[cp[0]]++;
            adjacency.get(cp[1]).add(cp[0]);
        }
        // Get all the courses with the indegree of 0.
        for(int i = 0; i < numCourses; i++)
            if(indegrees[i] == 0) queue.add(i);
        // BFS TopSort.
        while(!queue.isEmpty()) {
            int pre = queue.poll();
            numCourses--;
            for(int cur : adjacency.get(pre))
                if(--indegrees[cur] == 0) queue.add(cur);
        }
        return numCourses == 0;
    }
```

