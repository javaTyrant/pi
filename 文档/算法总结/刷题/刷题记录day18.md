[toc]

#[227. 基本计算器 II](https://leetcode-cn.com/problems/basic-calculator-ii/) 

>  实现一个基本的计算器来计算一个简单的字符串表达式的值。
>
>  字符串表达式仅包含非负整数，+， - ，*，/ 四种运算符和空格  。 整数除法仅保留整数部分。
>
>  示例 1:
>
>  输入: "3+2*2"
>  输出: 7
>  示例 2:
>
>  输入: " 3/2 "
>  输出: 1
>  示例 3:
>
>  输入: " 3+5 / 2 "
>  输出: 5
>  说明：
>
>  你可以假设所给定的表达式都是有效的。
>  请不要使用内置的库函数 eval。

```java
public int calculate(String s) {
        int[] array = new int[s.length()];
        char sign1 = ' ', sign2 = ' ';
        int i = 0,end = -1,tmp,start = 0;
        while(true) {
        	while(i<s.length()&&s.charAt(i)==' ') i++;
        	tmp = 0;
        	while(i<s.length()&&s.charAt(i)-'0'>=0) {
        		tmp = tmp*10+s.charAt(i)-'0';
        		i++;
        	}
        	if(sign2=='*')
        		array[end] *= tmp;
        	else if (sign2=='/')
        		array[end] /= tmp;
		else
			array[++end] = tmp;
        	while(i<s.length()&&s.charAt(i)==' ') i++;
        	if(i==s.length())
        		break;
        	sign2 = s.charAt(i);
        	if(s.charAt(i)=='+'||s.charAt(i)=='-') {
        		if(sign1=='+') {
        			array[start+1] += array[start];
        			start++;
        		}
        		else if(sign1=='-') {
        			array[start+1] = array[start]-array[start+1];
        			start++;
        		}
        		sign1 = s.charAt(i);
        	}
        	i++;
        }
        if(end==start)
        	return array[start];
        if(sign1=='+')
        	return array[start]+array[start+1];
        else
        	return array[start]-array[start+1];
    }
```



#[946. 验证栈序列](https://leetcode-cn.com/problems/validate-stack-sequences/) 

>  给定 pushed 和 popped 两个序列，每个序列中的 值都不重复，只有当它们可能是在最初空栈上进行的推入 push 和弹出 pop 操作序列的结果时，返回 true；否则，返回 false 。
>
>   
>
>  示例 1：
>
>  输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
>  输出：true
>  解释：我们可以按以下顺序执行：
>  push(1), push(2), push(3), push(4), pop() -> 4,
>  push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
>  示例 2：
>
>  输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
>  输出：false
>  解释：1 不能在 2 之前弹出。
>
>
>  提示：
>
>  0 <= pushed.length == popped.length <= 1000
>  0 <= pushed[i], popped[i] < 1000
>  pushed 是 popped 的排列。

```java
 public boolean validateStackSequences(int[] pushed, int[] popped) {
        Deque<Integer> stack = new ArrayDeque<>();
        int j = 0; //索引popped
        for (int i = 0; i < pushed.length; i++) {
            stack.push(pushed[i]);
            while (!stack.isEmpty() && stack.peek() == popped[j]) {
                j++;
                stack.pop();
            }
        }
        return stack.isEmpty();
    }
```



#[509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/) 

>  斐波那契数，通常用 F(n) 表示，形成的序列称为斐波那契数列。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是：
>
>  F(0) = 0,   F(1) = 1
>  F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
>  给定 N，计算 F(N)。
>
>   
>
>  示例 1：
>
>  输入：2
>  输出：1
>  解释：F(2) = F(1) + F(0) = 1 + 0 = 1.
>  示例 2：
>
>  输入：3
>  输出：2
>  解释：F(3) = F(2) + F(1) = 1 + 1 = 2.
>  示例 3：
>
>  输入：4
>  输出：3
>  解释：F(4) = F(3) + F(2) = 2 + 1 = 3.
>
>
>  提示：
>
>  0 ≤ N ≤ 30
>

```java
public int fib(int n) {
        int pre = 0;
        int cur = 1;
        int res = 0;
        for (int i = 0; i <= n; i++) {
            //其实只需要数组的两个元素
            res += pre;
            pre = cur;
            cur = res;
        }
        return res;
    }
```



#[137. 只出现一次的数字 II](https://leetcode-cn.com/problems/single-number-ii/) 

>  给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。
>
>  说明：
>
>  你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？
>
>  示例 1:
>
>  输入: [2,2,3,2]
>  输出: 3
>  示例 2:
>
>  输入: [0,1,0,1,0,1,99]
>  输出: 99

```java
 public int singleNumber(int[] nums) {
        int one = 0, two = 0, three;
        for (int num : nums) {
            // two的相应的位等于1，表示该位出现2次
            two |= (one & num);
            // one的相应的位等于1，表示该位出现1次
            one ^= num;
            // three的相应的位等于1，表示该位出现3次
            three = (one & two);
            // 如果相应的位出现3次，则该位重置为0
            two &= ~three;
            one &= ~three;
        }
        return one;
    }
```



#[237. 删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/) 

>  请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点。传入函数的唯一参数为 要被删除的节点 。
>
>   
>
>  现有一个链表 -- head = [4,5,1,9]，它可以表示为:
>
>  
>
>   
>
>  示例 1：
>
>  输入：head = [4,5,1,9], node = 5
>  输出：[4,1,9]
>  解释：给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
>  示例 2：
>
>  输入：head = [4,5,1,9], node = 1
>  输出：[4,5,9]
>  解释：给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
>
>
>  提示：
>
>  链表至少包含两个节点。
>  链表中所有节点的值都是唯一的。
>  给定的节点为非末尾节点并且一定是链表中的一个有效节点。
>  不要从你的函数中返回任何结果。

```java
public void deleteNode(ListNode node) {
        node.val = node.next.val;
        node.next = node.next.next;
    }
```



#[617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/) 

>  给定两个二叉树，想象当你将它们中的一个覆盖到另一个上时，两个二叉树的一些节点便会重叠。
>
>  你需要将他们合并为一个新的二叉树。合并的规则是如果两个节点重叠，那么将他们的值相加作为节点合并后的新值，否则不为 NULL 的节点将直接作为新二叉树的节点。
>
>  示例 1:
>
>  输入: 
>  	Tree 1                     Tree 2                  
>            1                         2                             
>           / \                       / \                            
>          3   2                     1   3                        
>         /                           \   \                      
>        5                             4   7                  
>  输出: 
>  合并后的树:
>  	     3
>  	    / \
>  	   4   5
>  	  / \   \ 
>  	 5   4   7
>  注意: 合并必须从两个树的根节点开始。

```java
public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if(t1 == null && t2 == null){
            return null;
        }
        if(t1 == null){
            return t2;
        }
        if(t2 == null){
            return t1;
        }
        t1.val = t1.val + t2.val;
        t1.left = mergeTrees(t1.left,t2.left);
        t1.right = mergeTrees(t1.right,t2.right);
        return t1;
    }
```



#[905. 按奇偶排序数组](https://leetcode-cn.com/problems/sort-array-by-parity/) 

>  给定一个非负整数数组 A，返回一个数组，在该数组中， A 的所有偶数元素之后跟着所有奇数元素。
>
>  你可以返回满足此条件的任何数组作为答案。
>
>   
>
>  示例：
>
>  输入：[3,1,2,4]
>  输出：[2,4,3,1]
>  输出 [4,2,3,1]，[2,4,1,3] 和 [4,2,1,3] 也会被接受。
>
>
>  提示：
>
>  1 <= A.length <= 5000
>  0 <= A[i] <= 5000

```java
 public int[] sortArrayByParity(int[] A) {
           int[] B = new int[A.length];
        int len = A.length - 1;
        int index = 0;
        for (int i = 0; i < A.length; i++) {
            if ((A[i] & 1) == 1) {
                B[len--] = A[i];
            } else
                B[index++] = A[i];
        }
        return B;
    }
```



#[258. 各位相加](https://leetcode-cn.com/problems/add-digits/) 

>  给定一个非负整数 num，反复将各个位上的数字相加，直到结果为一位数。
>
>  示例:
>
>  输入: 38
>  输出: 2 
>  解释: 各位相加的过程为：3 + 8 = 11, 1 + 1 = 2。 由于 2 是一位数，所以返回 2。
>  进阶:
>  你可以不使用循环或者递归，且在 O(1) 时间复杂度内解决这个问题吗？

```java
 public int addDigits(int num) {
         return (num-1)%9+1;
    }
```



#[212. 单词搜索 II](https://leetcode-cn.com/problems/word-search-ii/) 

>  给定一个 m x n 二维字符网格 board 和一个单词（字符串）列表 words，找出所有同时在二维网格和字典中出现的单词。
>
>  单词必须按照字母顺序，通过 相邻的单元格 内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母在一个单词中不允许被重复使用。
>
>   
>
>  示例 1：
>
>
>  输入：board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]], words = ["oath","pea","eat","rain"]
>  输出：["eat","oath"]
>  示例 2：
>
>
>  输入：board = [["a","b"],["c","d"]], words = ["abcb"]
>  输出：[]
>
>
>  提示：
>
>  m == board.length
>  n == board[i].length
>  1 <= m, n <= 12
>  board[i][j] 是一个小写英文字母
>  1 <= words.length <= 3 * 104
>  1 <= words[i].length <= 10
>  words[i] 由小写英文字母组成
>  words 中的所有字符串互不相同

```java
 public List<String> findWords(char[][] board, String[] words) {
        //构建字典树
        WordTrie myTrie = new WordTrie();
        TrieNode root = myTrie.root;
        //插入数据
        for (String word : words){
            myTrie.insert(word);
        }

        //构建结果集容器
        List<String> result = new LinkedList<>();
        //矩阵行数
        int m = board.length;
        //矩阵列数
        int n = board[0].length;
        //存储该节点是否访问
        boolean[][] visited = new boolean[n][m];
        //遍历整个二维数组
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                find(board, visited, i, j, m, n, result, root);
            }
        }
        return result;
    }

    private void find(char[][] board, boolean[][] visited, int i, int j, int m, int n, List<String> result, TrieNode cur) {
        //边界判断以及是否已经访问判断
        if (i < 0 || i >= m || j < 0 || j >= n || visited[j][i])
            return;
        //获取子节点状态，判断其是否有子节点
        cur = cur.child[board[i][j] - 'a'];
        if (cur == null) {
            return;
        }
        //修改节点状态，防止重复访问
        visited[j][i] = true;
        //找到单词加入
        if (cur.isEnd) {
            result.add(cur.val);
            //找到单词后，修改字典树内叶子节点状态为false，防止出现重复单词
            cur.isEnd = false;
        }
        find(board, visited, i + 1, j, m, n, result, cur);
        find(board, visited, i - 1, j, m, n, result, cur);
        find(board, visited, i, j + 1, m, n, result, cur);
        find(board, visited, i, j - 1, m, n, result, cur);
        //最后修改节点状态为未访问状态
        visited[j][i] = false;
    }


    /**
     * 字典树
     */
    class WordTrie {
        //创建根节点
        TrieNode root = new TrieNode();

        void insert(String s) {
            TrieNode cur = root;
            for (char c : s.toCharArray()) {
                //判断是否存在该字符的节点，不存在则创建
                if (cur.child[c - 'a'] == null) {
                    cur.child[c - 'a'] = new TrieNode();
                    cur = cur.child[c - 'a'];
                } else
                    cur = cur.child[c - 'a'];
            }
            //遍历结束后，修改叶子节点的状态，并存储字符串
            cur.isEnd = true;
            cur.val = s;
        }
    }

    /**
     * 字典树节点
     */
    class TrieNode {
        /**
         * 存储最后节点的字符串
         */
        String val;
        /**
         * 根据字符排序，[a,b,c,……,z]
         */
        TrieNode[] child = new TrieNode[26];
        /**
         * 是否是最后叶子节点
         */
        boolean isEnd = false;
    }
```

```java
public List<String> findWords(char[][] board, String[] words) {
        List<String> results = new ArrayList<>();
        for (String word : words) {
            if (exist(board, word)) {
                results.add(word);
            }
        }
        return results;
    }

     private boolean exist(char[][] board, String word) {
        char[] chars = word.toCharArray();
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (dfs(board, chars, i, j, 0)) {
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, char[] word, int i, int j, int index) {
        //边界的判断，如果越界直接返回false。index表示的是查找到字符串word的第几个字符，
        //如果这个字符不等于board[i][j]，说明验证这个坐标路径是走不通的，直接返回false
        if (i >= board.length || i < 0 || j >= board[0].length || j < 0 || board[i][j] != word[index]) {
            return false;
        }
        //如果word的每个字符都查找完了，直接返回true
        if (index == word.length - 1) {
            return true;
        }
        //把当前坐标的值保存下来，为了在最后复原
        char tmp = board[i][j];
        //然后修改当前坐标的值
        board[i][j] = '.';
        //走递归，沿着当前坐标的上下左右4个方向查找
        boolean res = dfs(board, word, i + 1, j, index + 1) || dfs(board, word, i - 1, j, index + 1)
                || dfs(board, word, i, j + 1, index + 1) || dfs(board, word, i, j - 1, index + 1);
        //递归之后再把当前的坐标复原
        board[i][j] = tmp;
        return res;
    }
```



#[201. 数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/) 

>  给定范围 [m, n]，其中 0 <= m <= n <= 2147483647，返回此范围内所有数字的按位与（包含 m, n 两端点）。
>
>  示例 1: 
>
>  输入: [5,7]
>  输出: 4
>  示例 2:
>
>  输入: [0,1]
>  输出: 0

```java
public int rangeBitwiseAnd(int m, int n) {
        int offset = 0;
        for (; m != n; ++offset) {
         m >>= 1;
         n >>= 1;
        }
        return n << offset;
    }
```



#[301. 删除无效的括号](https://leetcode-cn.com/problems/remove-invalid-parentheses/) 

>  
>  删除最小数量的无效括号，使得输入的字符串有效，返回所有可能的结果。
>
>  **说明:** 输入可能包含了除 `(` 和 `)` 以外的字符。
>
>  **示例 1:**
>
>  ```
>  输入: "()())()"
>  输出: ["()()()", "(())()"]
>  ```
>
>  **示例 2:**
>
>  ```
>  输入: "(a)())()"
>  输出: ["(a)()()", "(a())()"]
>  ```
>
>  **示例 3:**
>
>  ```sql
>  输入: ")("
>  输出: [""]
>  ```

```java
public List<String> removeInvalidParentheses(String s) {
        Set<String> set = new HashSet<>();
        List<String> res = new ArrayList<>();
        Queue<String> queue = new LinkedList<>();
        queue.offer(s);
        set.add(s);
        boolean exist = false;
        while (!queue.isEmpty()) {
            String currentString = queue.poll();
            if (isValid(currentString)) {
                res.add(currentString);
                exist = true;
            }
            if (exist) {
                continue;
            }
            for (int i = 0; i < currentString.length(); i++) {
                if (!Character.isLetter(currentString.charAt(i))) {
                    String newString = currentString.substring(0, i) + currentString.substring(i + 1);
                    if (!set.contains(newString)) {
                        queue.offer(newString);
                        set.add(newString);
                    }
                }
            }
        }
        return res;
    }

    private boolean isValid(String str) {
        int count = 0;
        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) == '(') {
                count++;
            } else if (str.charAt(i) == ')') {
                if (count == 0) {
                    return false;
                } else {
                    count--;
                }
            }
        }
        return count == 0;
    }
```



#[611. 有效三角形的个数](https://leetcode-cn.com/problems/valid-triangle-number/) 

>  给定一个包含非负整数的数组，你的任务是统计其中可以组成三角形三条边的三元组个数。
>
>  示例 1:
>
>  输入: [2,2,3,4]
>  输出: 3
>  解释:
>  有效的组合是: 
>  2,3,4 (使用第一个 2)
>  2,3,4 (使用第二个 2)
>  2,2,3
>  注意:
>
>  数组长度不超过1000。
>  数组里整数的范围为 [0, 1000]。

```java
  public int triangleNumber(int[] nums) {
         Arrays.sort(nums);
        int result = 0;
        for (int i = nums.length - 1; i >= 2; i--) {
            int k = 0;
            int j = i - 1;
            while (k < j) {
                //满足该条件，说明从num[k]到num[j]的数都满足要求，结果直接加上j - k
                if (nums[k] + nums[j] > nums[i]) {
                    result += j - k;
                    j--;
                } else {
                    //否则k自增，重新判断
                    k++;
                }
            }
        }
        return result;
    }
```



#[剑指 Offer 35 - 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/) 

>  请实现 `copyRandomList` 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 `next` 指针指向下一个节点，还有一个 `random` 指针指向链表中的任意节点或者 `null`。

```java
  //推荐双百方法一：指针版  需要一个辅助的map，下面还有一个调用方法的版本，不需要额外创建指针
   Map<Node,Node> map1 = new HashMap<>();//map前面保存原链表节点，后面保存复制节点，则可通过前面的key直接找到后面的value
   public Node copyRandomList(Node head) {
       if(head==null) return null;
       Node res = new Node(head.val);
       map1.put(head,res);
       Node tmpR = res;
       Node tmpH = head;
       while(tmpH.next!=null){
           tmpR.next = new Node(tmpH.next.val);
           tmpR=tmpR.next;
           tmpH=tmpH.next;
           map1.put(tmpH,tmpR);
       }
       tmpR = res; 
       tmpH = head;
       while(tmpH!=null){
           if(tmpH.random!=null) tmpR.random = map1.get(tmpH.random);
           tmpH = tmpH.next;
           tmpR = tmpR.next; 
       }
       return res; 
   }
```



#[871. 最低加油次数](https://leetcode-cn.com/problems/minimum-number-of-refueling-stops/) 

>  汽车从起点出发驶向目的地，该目的地位于出发位置东面 target 英里处。
>
>  沿途有加油站，每个 station[i] 代表一个加油站，它位于出发位置东面 station[i][0] 英里处，并且有 station[i][1] 升汽油。
>
>  假设汽车油箱的容量是无限的，其中最初有 startFuel 升燃料。它每行驶 1 英里就会用掉 1 升汽油。
>
>  当汽车到达加油站时，它可能停下来加油，将所有汽油从加油站转移到汽车中。
>
>  为了到达目的地，汽车所必要的最低加油次数是多少？如果无法到达目的地，则返回 -1 。
>
>  注意：如果汽车到达加油站时剩余燃料为 0，它仍然可以在那里加油。如果汽车到达目的地时剩余燃料为 0，仍然认为它已经到达目的地。
>
>   
>
>  示例 1：
>
>  输入：target = 1, startFuel = 1, stations = []
>  输出：0
>  解释：我们可以在不加油的情况下到达目的地。
>  示例 2：
>
>  输入：target = 100, startFuel = 1, stations = [[10,100]]
>  输出：-1
>  解释：我们无法抵达目的地，甚至无法到达第一个加油站。
>  示例 3：
>
>  输入：target = 100, startFuel = 10, stations = [[10,60],[20,30],[30,30],[60,40]]
>  输出：2
>  解释：
>  我们出发时有 10 升燃料。
>  我们开车来到距起点 10 英里处的加油站，消耗 10 升燃料。将汽油从 0 升加到 60 升。
>  然后，我们从 10 英里处的加油站开到 60 英里处的加油站（消耗 50 升燃料），
>  并将汽油从 10 升加到 50 升。然后我们开车抵达目的地。
>  我们沿途在1两个加油站停靠，所以返回 2 。
>
>
>  提示：
>
>  1 <= target, startFuel, stations[i][1] <= 10^9
>  0 <= stations.length <= 500
>  0 < stations[0][0] < stations[1][0] < ... < stations[stations.length-1][0] < target

```java
 public int minRefuelStops(int target, int startFuel, int[][] stations) {

        //这一题就是优先队列的应用，太扯淡了
        //完全就是特题特解，别的方法都相形见绌
        //优先队列可以是nlogn的解法，这太霸道了
        
        if(stations.length == 0)
            return startFuel>=target?0:-1;
        PriorityQueue<Integer>queue = new PriorityQueue<Integer>((o1,o2)->{
            return o2-o1;
        });
        int sum = startFuel;
        int ans = 0;
        for(int i = 0;i < stations.length;i++)
        {
            while(sum < stations[i][0])
            {
                Integer ii = queue.poll();
                if(ii == null)return -1;
                sum += ii;
                ans++;
            }
            queue.offer(stations[i][1]);
        }
        while(sum < target)
            {
                Integer ii = queue.poll();
                if(ii == null)return -1;
                sum += ii;
                ans++;
            }
        return ans;
    }
```



#[面试题 02.02 - 返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/) 

>  实现一种算法，找出单向链表中倒数第 k 个节点。返回该节点的值。
>
>  注意：本题相对原题稍作改动
>
>  示例：
>
>  输入： 1->2->3->4->5 和 k = 2
>  输出： 4
>  说明：
>
>  给定的 k 保证是有效的。
>

```java
 public int kthToLast(ListNode head, int k) {
        ListNode fast = head;
        ListNode slow = head;
        while(k != 0){
            fast = fast.next;
            k--;
        }
        while(fast != null){
            fast = fast.next;
            slow = slow.next;
        }
        return slow.val;
    }
```



#[剑指 Offer 54 - 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/) 

>  
>  给定一棵二叉搜索树，请找出其中第k大的节点。
>
>   
>
>  **示例 1:**
>
>  ```
>  输入: root = [3,1,4,null,2], k = 1
>     3
>    / \
>   1   4
>    \
>     2
>  输出: 4
>  ```
>
>  **示例 2:**
>
>  ```
>  输入: root = [5,3,6,2,4,null,null,1], k = 3
>         5
>        / \
>       3   6
>      / \
>     2   4
>    /
>   1
>  输出: 4
>  ```

```java
  private int ans = 0, cnt = 0;
    public int kthLargest(TreeNode root, int k) {
        dfs(root, k);
        return ans;
    }
    private void dfs(TreeNode root, int k) {
        if(root == null) return ;
        dfs(root.right, k);
        if(++cnt == k) {
            ans = root.val;
        }
        dfs(root.left, k);
    }
```



#[剑指 Offer 59 - I - 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/) 

>  给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值。
>
>  示例:
>
>  输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
>  输出: [3,3,5,5,6,7] 
>  解释: 
>
>    滑动窗口的位置                最大值
>  ---------------               -----
>  [1  3  -1] -3  5  3  6  7       3
>   1 [3  -1  -3] 5  3  6  7       3
>   1  3 [-1  -3  5] 3  6  7       5
>   1  3  -1 [-3  5  3] 6  7       5
>   1  3  -1  -3 [5  3  6] 7       6
>   1  3  -1  -3  5 [3  6  7]      7
>
>
>  提示：
>
>  你可以假设 k 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小。
>

```java
public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return new int[0];
        }
        int[] res = new int[nums.length - k + 1];
        Deque<Integer> queue = new ArrayDeque<>();
        for(int i = 0, j = 0; i < nums.length; i++) {
            if(!queue.isEmpty() && i - queue.peek() >= k) {
                queue.poll();
            }
            while(!queue.isEmpty() && nums[i] > nums[queue.peekLast()]) {
                queue.pollLast();
            }
            queue.offer(i);
            if(i >= k - 1) {
                res[j++] = nums[queue.peek()];
            }
        }
        return res;
    }
```



#[230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/) 

>  给定一个二叉搜索树，编写一个函数 kthSmallest 来查找其中第 k 个最小的元素。
>
>  说明：
>  你可以假设 k 总是有效的，1 ≤ k ≤ 二叉搜索树元素个数。
>
>  示例 1:
>
>  输入: root = [3,1,4,null,2], k = 1
>     3
>    / \
>   1   4
>    \
>     2
>  输出: 1
>  示例 2:
>
>  输入: root = [5,3,6,2,4,null,null,1], k = 3
>         5
>        / \
>       3   6
>      / \
>     2   4
>    /
>   1
>  输出: 3

```java
  public int kthSmallest(TreeNode root, int k) {
        midOrder(root,k);
        return res;
    }
     private int i=1;
    private int res;
    private void midOrder(TreeNode root,int k){
        if (root==null)return;
        midOrder(root.left,k);
      	//相当于设置了一个返回的标记，表示已经找到第k小的值了，不用继续向下寻找了。
        if(i>k)return ;
        if(i++==k){
            res=root.val;
            return;
        }
        midOrder(root.right,k);
    }
```

