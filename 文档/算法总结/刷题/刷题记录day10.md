[toc]

# [97. 交错字符串](https://leetcode-cn.com/problems/interleaving-string/)

>   给定三个字符串 s1、s2、s3，请你帮忙验证 s3 是否是由 s1 和 s2 交错 组成的。
>
>   两个字符串 s 和 t 交错 的定义与过程如下，其中每个字符串都会被分割成若干 非空 子字符串：
>
>   s = s1 + s2 + ... + sn
>   t = t1 + t2 + ... + tm
>   |n - m| <= 1
>   交错 是 s1 + t1 + s2 + t2 + s3 + t3 + ... 或者 t1 + s1 + t2 + s2 + t3 + s3 + ...
>   提示：a + b 意味着字符串 a 和 b 连接。
>

```java
public boolean isInterleave(String s1, String s2, String s3) {
        int n = s1.length(), m = s2.length(), t = s3.length();
        
        if (n + m != t) {
            return false;
        }

        boolean[][] f = new boolean[n + 1][m + 1];

        f[0][0] = true;
        for (int i = 0; i <= n; ++i) {
            for (int j = 0; j <= m; ++j) {
                int p = i + j - 1;
                if (i > 0) {
                    f[i][j] = f[i][j] || (f[i - 1][j] && s1.charAt(i - 1) == s3.charAt(p));
                }
                if (j > 0) {
                    f[i][j] = f[i][j] || (f[i][j - 1] && s2.charAt(j - 1) == s3.charAt(p));
                }
            }
        }
        return f[n][m];
    }
```

# [151. 翻转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)

>   给定一个字符串，逐个翻转字符串中的每个单词。
>
>   说明：
>
>   无空格字符构成一个 单词 。
>   输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
>   如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
>
>
>   示例 1：
>
>   输入："the sky is blue"
>   输出："blue is sky the"
>   示例 2：
>
>   输入："  hello world!  "
>   输出："world! hello"
>   解释：输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
>   示例 3：
>
>   输入："a good   example"
>   输出："example good a"
>   解释：如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
>   示例 4：
>
>   输入：s = "  Bob    Loves  Alice   "
>   输出："Alice Loves Bob"
>   示例 5：
>
>   输入：s = "Alice does not even like bob"
>   输出："bob like even not does Alice"

```java
 public String reverseWords(String s) {
     int length = s.length();
        if (s == null || length == 0) return s;
        //相当于指针
        int end = length;
        StringBuilder ret = new StringBuilder();
        for (int i = length - 1; i >= 0; i--) {
            //何解?如果是空格指针后退一格,重新执行循环,妙用指针
            if (s.charAt(i) == ' ') end = i;
                //如果退到首位或者之前是空格,相当于拉了一个end-i的窗格
            else if (i == 0 || s.charAt(i - 1) == ' ') {
                //如果里面有单词，新加单词时先加空格
                if (ret.length() != 0)
                    ret.append(' ');
                //添加窗口
                ret.append(s, i, end);
            }
        }
        return ret.toString();
    }
```

# [77. 组合](https://leetcode-cn.com/problems/combinations/)

>   给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。
>
>   示例:
>
>   输入: n = 4, k = 2
>   输出:
>   [
>     [2,4],
>     [3,4],
>     [2,3],
>     [1,2],
>     [1,3],
>     [1,4],
>   ]

```java
   public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        dfs(res, list, n, k, 1);
        return res;
    }

    private void dfs(List<List<Integer>> res, List<Integer> list, int n, int k, int start) {
        if (list.size() == k) {
            res.add(new ArrayList<>(list));
        }
        for (int i = start; i <= n; i++) {
            list.add(i);
            dfs(res, list, n, k, i + 1);
            list.remove(list.size() - 1);
        }
    }
```

# [剑指 Offer 05 - 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

>   请实现一个函数，把字符串 s 中的每个空格替换成"%20"。
>
>    
>
>   示例 1：
>
>   输入：s = "We are happy."
>   输出："We%20are%20happy."

```java
 public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for(int i = 0 ; i < s.length(); i++){
            char c = s.charAt(i);
            if(c == ' ') sb.append("%20");
            else sb.append(c);
        }
        return sb.toString();
    }
```

# [剑指 Offer 48 - 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

>   请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。
>
>    
>
>   示例 1:
>
>   输入: "abcabcbb"
>   输出: 3 
>   解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
>   示例 2:
>
>   输入: "bbbbb"
>   输出: 1
>   解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
>   示例 3:
>
>   输入: "pwwkew"
>   输出: 3
>   解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
>        请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。

```java
 public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        Map<Character, Integer> map = new HashMap<>();
        for (int end = 0, start = 0; end < n; end++) {
            char alpha = s.charAt(end);
            if (map.containsKey(alpha)) {
                start = Math.max(map.get(alpha), start);
            }
            ans = Math.max(ans, end - start + 1);
            //为什么end + 1
            map.put(s.charAt(end), end + 1);
        }
        return ans;
    }
```

#[74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/) 

>   编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：
>
>   每行中的整数从左到右按升序排列。
>   每行的第一个整数大于前一行的最后一个整数。
>
>   ![img](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)

```java
   public boolean searchMatrix(int[][] matrix, int target) {
        if (matrix.length == 0)
            return false;
        int row = 0, col = matrix[0].length - 1;
        while (row < matrix.length && col >= 0) {
            if (matrix[row][col] < target)
                row++;
            else if (matrix[row][col] > target)
                col--;
            else
                return true;
        }
        return false;
    }
```

# [315. 计算右侧小于当前元素的个数](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/)

>   给定一个整数数组 nums，按要求返回一个新数组 counts。数组 counts 有该性质： counts[i] 的值是  nums[i] 右侧小于 nums[i] 的元素的数量。
>
>    
>
>   示例：
>
>   输入：nums = [5,2,6,1]
>   输出：[2,1,1,0] 
>   解释：
>   5 的右侧有 2 个更小的元素 (2 和 1)
>   2 的右侧仅有 1 个更小的元素 (1)
>   6 的右侧有 1 个更小的元素 (1)
>   1 的右侧有 0 个更小的元素
>
>
>   提示：
>
>   0 <= nums.length <= 10^5
>   -10^4 <= nums[i] <= 10^4

```java
public List<Integer> countSmaller(int[] nums) {
        if (nums.length == 0) {
            return new ArrayList<>();
        }
        int min = Integer.MAX_VALUE; // nums数组最小值
        for (int value : nums) {
            if (value < min) {
                min = value;
            }
        }
        for (int i = 0; i < nums.length; i++) {
            nums[i] = nums[i] - min + 1;
        }
        
        int max = Integer.MIN_VALUE;
        for (int value : nums) {
            if (value > max) {
                max = value;
            }
        }
        
        int[] BITree = new int[max + 1];
        BITree[0] = 0;
        int[] countArr = new int[nums.length];
        for (int i = nums.length - 1; i >= 0; i--) {
            int count = getSum(nums[i] - 1, BITree);
            countArr[i] = count;
            update(nums[i], BITree);
        }
        List<Integer> result = new ArrayList<>();
        for (int value : countArr) {
            result.add(value);
        }
        return result;
    }
    
    public static int getSum(int value, int[] BITree) { // 获得a[i]从1，value的和
        int sum = 0;
        while (value > 0) {
            sum += BITree[value];
            value -= (value & -value);
        }
        return sum;
    }

    public static void update(int value, int[] BITree) {
        while (value <= BITree.length - 1) {
            BITree[value] += 1;
            value += (value & -value);
        }
    }
```

# [670. 最大交换](https://leetcode-cn.com/problems/maximum-swap/)

>   
>   给定一个非负整数，你**至多**可以交换一次数字中的任意两位。返回你能得到的最大值。
>
>   **示例 1 :**
>
>   ```
>   输入: 2736
>   输出: 7236
>   解释: 交换数字2和数字7。
>   ```
>
>   **示例 2 :**
>
>   ```
>   输入: 9973
>   输出: 9973
>   解释: 不需要交换。
>   ```
>
>   **注意:**
>
>   1. 给定数字的范围是 [0, 108]

```java
public int maximumSwap(int num) {
    char[] chars = (num + "").toCharArray();
        if (chars.length == 1) return num;
        HashMap<Character, Integer> map = new HashMap<>();
        for (int i = 0; i < chars.length; i++) {
            map.put(chars[i], i);
        }
        Character[] clone = new Character[chars.length];
        for (int i = 0; i < clone.length; i++) clone[i] = chars[i];
        Arrays.sort(clone, (o1, o2) -> o2 - o1);
        for (int i = 0; i < chars.length; i++) {
            if (chars[i] != clone[i]) {
                Integer cloneIndex = map.get(clone[i]);
                char temp = chars[i];
                chars[i] = clone[i];
                chars[cloneIndex] = temp;
                break;
            }
        }
        return Integer.parseInt(new String(chars));
    }
```

# [111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

>   给定一个二叉树，找出其最小深度。
>
>   最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
>
>   **说明：**叶子节点是指没有子节点的节点。

```java
  public int minDepth(TreeNode root) {
        if (root == null) return 0;
        if (root.left == null && root.right == null) return 1;
        else if (root.left == null && root.right != null) return 1 + minDepth(root.right); 
        else if (root.left != null && root.right == null) return 1 + minDepth(root.left);
        else return 1 + Math.min(minDepth(root.left), minDepth(root.right));
    }
```

# [887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

>   你将获得 K 个鸡蛋，并可以使用一栋从 1 到 N  共有 N 层楼的建筑。
>
>   每个蛋的功能都是一样的，如果一个蛋碎了，你就不能再把它掉下去。
>
>   你知道存在楼层 F ，满足 0 <= F <= N 任何从高于 F 的楼层落下的鸡蛋都会碎，从 F 楼层或比它低的楼层落下的鸡蛋都不会破。
>
>   每次移动，你可以取一个鸡蛋（如果你有完整的鸡蛋）并把它从任一楼层 X 扔下（满足 1 <= X <= N）。
>
>   你的目标是确切地知道 F 的值是多少。
>
>   无论 F 的初始值如何，你确定 F 的值的最小移动次数是多少？
>
>    
>
>   示例 1：
>
>   输入：K = 1, N = 2
>   输出：2
>   解释：
>   鸡蛋从 1 楼掉落。如果它碎了，我们肯定知道 F = 0 。
>   否则，鸡蛋从 2 楼掉落。如果它碎了，我们肯定知道 F = 1 。
>   如果它没碎，那么我们肯定知道 F = 2 。
>   因此，在最坏的情况下我们需要移动 2 次以确定 F 是多少。
>   示例 2：
>
>   输入：K = 2, N = 6
>   输出：3
>   示例 3：
>
>   输入：K = 3, N = 14
>   输出：4
>
>
>   提示：
>
>   1 <= K <= 100
>   1 <= N <= 10000

```java
  public int superEggDrop(int K, int N) {
        int[] dp = new int[K + 1];
        int m = 0;
        while (dp[K] < N){
            for (int i = K; i >= 1; i--){
                dp[i] += dp[i - 1] + 1;
            }
            m++;
        }
        return m;
    }
```

#[剑指 Offer 34 - 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

>   输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。
>
>    
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
 private List<List<Integer>> res;

    // 二叉树中和为某一值的路径
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        res = new ArrayList<>();
        backtrack(root, sum, new ArrayList<>());
        return res;
    }

    private void backtrack(TreeNode node, int target, List<Integer> collector) {
        if (node == null) {
            return;
        }
        collector.add(node.val);
        target -= node.val;
        if (target == 0 && node.left == null && node.right == null) {
            res.add(new ArrayList<>(collector));
        } else {
            backtrack(node.left, target, collector);
            backtrack(node.right, target, collector);
        }
        collector.remove(collector.size() - 1);
    }
```

# [40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

>   给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。
>
>   candidates 中的每个数字在每个组合中只能使用一次。
>
>   说明：
>
>   所有数字（包括目标数）都是正整数。
>   解集不能包含重复的组合。 
>   示例 1:
>
>   输入: candidates = [10,1,2,7,6,1,5], target = 8,
>   所求解集为:
>   [
>     [1, 7],
>     [1, 2, 5],
>     [2, 6],
>     [1, 1, 6]
>   ]
>   示例 2:
>
>   输入: candidates = [2,5,2,1,2], target = 5,
>   所求解集为:
>   [
>     [1,2,2],
>     [5]
>   ]

```java
  public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(candidates);
        help(res,candidates,target,candidates.length,new ArrayList<>(),0);
        return res;
    }
    private void help(List<List<Integer>> res,int[] candidates,int target,int len,List<Integer> list,int begin){
        if(target == 0){
            res.add(new ArrayList<>(list));
        }
        for(int i = begin;i<len;i++){
            if(target - candidates[i]<0) return;
            if(i > begin && candidates[i] == candidates[i-1])  continue;
            list.add(candidates[i]);
            //用i,不能用begin哦
            help(res,candidates,target-candidates[i],len,list,i+1);
            list.remove(list.size()-1);
        }
    }
```

# [726. 原子的数量](https://leetcode-cn.com/problems/number-of-atoms/)

>   给定一个化学式formula（作为字符串），返回每种原子的数量。
>
>   原子总是以一个大写字母开始，接着跟随0个或任意个小写字母，表示原子的名字。
>
>   如果数量大于 1，原子后会跟着数字表示原子的数量。如果数量等于 1 则不会跟数字。例如，H2O 和 H2O2 是可行的，但 H1O2 这个表达是不可行的。
>
>   两个化学式连在一起是新的化学式。例如 H2O2He3Mg4 也是化学式。
>
>   一个括号中的化学式和数字（可选择性添加）也是化学式。例如 (H2O2) 和 (H2O2)3 是化学式。
>
>   给定一个化学式，输出所有原子的数量。格式为：第一个（按字典序）原子的名子，跟着它的数量（如果数量大于 1），然后是第二个原子的名字（按字典序），跟着它的数量（如果数量大于 1），以此类推。
>
>   示例 1:
>
>   输入: 
>   formula = "H2O"
>   输出: "H2O"
>   解释: 
>   原子的数量是 {'H': 2, 'O': 1}。
>   示例 2:
>
>   输入: 
>   formula = "Mg(OH)2"
>   输出: "H2MgO2"
>   解释: 
>   原子的数量是 {'H': 2, 'Mg': 1, 'O': 2}。
>   示例 3:
>
>   输入: 
>   formula = "K4(ON(SO3)2)2"
>   输出: "K4N2O14S4"
>   解释: 
>   原子的数量是 {'K': 4, 'N': 2, 'O': 14, 'S': 4}。
>   注意:
>
>   所有原子的第一个字母为大写，剩余字母都是小写。
>   formula的长度在[1, 1000]之间。
>   formula只包含字母、数字和圆括号，并且题目中给定的是合法的化学式。

```java
public String countOfAtoms(String formula) {
        int N = formula.length();
        Stack<Map<String, Integer>> stack = new Stack();
        stack.push(new TreeMap());

        for (int i = 0; i < N;) {
            if (formula.charAt(i) == '(') {
                stack.push(new TreeMap());
                i++;
            } else if (formula.charAt(i) == ')') {
                Map<String, Integer> top = stack.pop();
                int iStart = ++i, multiplicity = 1;
                while (i < N && Character.isDigit(formula.charAt(i))) i++;
                if (i > iStart) multiplicity = Integer.parseInt(formula.substring(iStart, i));
                for (String c: top.keySet()) {
                    int v = top.get(c);
                    stack.peek().put(c, stack.peek().getOrDefault(c, 0) + v * multiplicity);
                }
            } else {
                int iStart = i++;
                while (i < N && Character.isLowerCase(formula.charAt(i))) i++;
                String name = formula.substring(iStart, i);
                iStart = i;
                while (i < N && Character.isDigit(formula.charAt(i))) i++;
                int multiplicity = i > iStart ? Integer.parseInt(formula.substring(iStart, i)) : 1;
                stack.peek().put(name, stack.peek().getOrDefault(name, 0) + multiplicity);
            }
        }

        StringBuilder ans = new StringBuilder();
        for (String name: stack.peek().keySet()) {
            ans.append(name);
            int multiplicity = stack.peek().get(name);
            if (multiplicity > 1) ans.append("" + multiplicity);
        }
        return new String(ans);
    }
```

# [208. 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

>   实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。
>
>   示例:
>
>   Trie trie = new Trie();
>
>   trie.insert("apple");
>   trie.search("apple");   // 返回 true
>   trie.search("app");     // 返回 false
>   trie.startsWith("app"); // 返回 true
>   trie.insert("app");   
>   trie.search("app");     // 返回 true
>   说明:
>
>   你可以假设所有的输入都是由小写字母 a-z 构成的。
>   保证所有输入均为非空字符串。

```java
 private TrieNode node;

    private class TrieNode{
        private TrieNode[] next;
        private boolean isEnd;
        //构造器勿忘
        public TrieNode(){
            next = new TrieNode[26];
            isEnd = false;
        }
    }
    /** Initialize your data structure here. */
    public Trie() {
        node = new TrieNode();
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        TrieNode tmp = node;
        for(char c : word.toCharArray()){
            int n = c - 'a';
            if(tmp.next[n] == null){
                tmp.next[n] = new TrieNode();
            }
            tmp = tmp.next[n];
        }
        tmp.isEnd = true;
    }
    
    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        TrieNode tmp = node;
        for(char c : word.toCharArray()){
            int n = c - 'a';
            if(tmp.next[n] == null) return false;
            tmp = tmp.next[n];
        }
        return tmp.isEnd;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
          TrieNode tmp = node;
        for(char c : prefix.toCharArray()){
            int n = c - 'a';
            if(tmp.next[n] == null) return false;
            tmp = tmp.next[n];
        }
        return true;
    }
```

# [225. 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

>   使用队列实现栈的下列操作：
>
>   push(x) -- 元素 x 入栈
>   pop() -- 移除栈顶元素
>   top() -- 获取栈顶元素
>   empty() -- 返回栈是否为空
>   注意:
>
>   你只能使用队列的基本操作-- 也就是 push to back, peek/pop from front, size, 和 is empty 这些操作是合法的。
>   你所使用的语言也许不支持队列。 你可以使用 list 或者 deque（双端队列）来模拟一个队列 , 只要是标准的队列操作即可。
>   你可以假设所有操作都是有效的（例如, 对一个空的栈不会调用 pop 或者 top 操作）。

```java
class MyStack {
   //用队列实现栈
    private Queue<Integer> q1 = new LinkedList<>();
    private Queue<Integer> q2 = new LinkedList<>();
    private int top;

    /** Initialize your data structure here. */
    public MyStack() {

    }
    
    /** Push element x onto stack. */
    public void push(int x) {
        q1.add(x);
        top = x;
    }
    
    /** Removes the element on top of the stack and returns that element. */
    public int pop() {
         while (q1.size() > 1) {
            top = q1.remove();
            q2.add(top);
        }
        int t = q1.remove();
        Queue<Integer> temp = q1;
        q1 = q2;
        q2 = temp;
        return t;
    }
    
    /** Get the top element. */
    public int top() {
        return top;
    }
    
    /** Returns whether the stack is empty. */
    public boolean empty() {
        return q1.size() == 0;
    }
}
```

# [138. 复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

>   给定一个链表，每个节点包含一个额外增加的随机指针，该指针可以指向链表中的任何节点或空节点。
>
>   要求返回这个链表的 深拷贝。 
>
>   我们用一个由 n 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 [val, random_index] 表示：
>
>   val：一个表示 Node.val 的整数。
>   random_index：随机指针指向的节点索引（范围从 0 到 n-1）；如果不指向任何节点，则为  null 。

```java
 public Node copyRandomList(Node head) {
         if(head == null)
            return null;
        Node cur = head;
        while(cur != null) {
            Node newCur = new Node(cur.val);
            newCur.next = cur.next;
            cur.next = newCur;
            cur = newCur.next;
        }
        cur = head;
        while(cur != null) {
            if(cur.next != null && cur.random != null)
                cur.next.random = cur.random.next;
            cur = cur.next.next;
        }
        cur = head;
        Node res = head.next;
        Node tcur = head.next;
        while(cur != null) {
            cur.next = cur.next.next;
            cur = cur.next;
            if(cur != null) {
                tcur.next = tcur.next.next;
                tcur = tcur.next;
            }
        }
        return res;
    }
```

# [718. 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

>   给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度。
>
>    
>
>   示例：
>
>   输入：
>   A: [1,2,3,2,1]
>   B: [3,2,1,4,7]
>   输出：3
>   解释：
>   长度最长的公共子数组是 [3, 2, 1] 。
>
>
>   提示：
>
>   1 <= len(A), len(B) <= 1000

```java
  public int findLength(int[] A, int[] B) {
        if (A == null || B == null) {
            return 0;
        }
        int res = 0;
        int[][] dp = new int[A.length + 1][B.length + 1];
        for (int i = 1; i < dp.length; i++) {
            for (int j = 1; j < dp[i].length; j++) {
                dp[i][j] = A[i - 1] == B[j - 1] ? dp[i - 1][j - 1] + 1 : 0;
                res = Math.max(res, dp[i][j]);
            }
        }
        return res;
    }
```

#[647. 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

>   
>   给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。
>
>   具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。
>
>   **示例 1：**
>
>   ```
>   输入："abc"
>   输出：3
>   解释：三个回文子串: "a", "b", "c"
>   ```
>
>   **示例 2：**
>
>   ```
>   输入："aaa"
>   输出：6
>   解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"
>   ```
>
>    
>
>   **提示：**
>
>   - 输入的字符串长度不会超过 1000 。

```java
  	int num = 0;
    public int countSubstrings(String s) {
        for (int i=0; i < s.length(); i++){
            count(s, i, i);//回文串长度为奇数
            count(s, i, i+1);//回文串长度为偶数
        }
        return num;
    }
    
    public void count(String s, int start, int end){
        while(start >= 0 && end < s.length() && s.charAt(start) == s.charAt(end)){
            num++;
            start--;
            end++;
        }
    }
```

