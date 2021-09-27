[toc]

#[剑指 Offer 32 - I - 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/) 

>  从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。
>
>   
>
>  例如:
>  给定二叉树: [3,9,20,null,null,15,7],
>
>      3
>     / \
>    9  20
>      /  \
>     15   7
>  返回：
>
>  [3,9,20,15,7]
>
>
>  提示：
>
>  节点总数 <= 1000
>

```java
  public int[] levelOrder(TreeNode root) {
        if(root == null) return new int[0];
        List<Integer> res = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
           TreeNode current =  queue.poll();
           res.add(current.val);
           if(current.left != null) queue.add(current.left);
           if(current.right != null) queue.add(current.right);
        }
        int[] r = new int[res.size()];
        for(int i = 0;i<res.size();i++){
            r[i] = res.get(i);
        }
        return r;
    }
```



#[剑指 Offer 59 - II - 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/) 

>  请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)。
>
>  若队列为空，pop_front 和 max_value 需要返回 -1
>
>  示例 1：
>
>  输入: 
>  ["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]
>  [[],[1],[2],[],[],[]]
>  输出: [null,null,null,2,1,2]
>  示例 2：
>
>  输入: 
>  ["MaxQueue","pop_front","max_value"]
>  [[],[],[]]
>  输出: [null,-1,-1]
>
>
>  限制：
>
>  1 <= push_back,pop_front,max_value的总操作数 <= 10000
>  1 <= value <= 10^5

```java
class MaxQueue {

    private Node max;
    private Node root;
    private Node tail;
    class Node{
        int val;
        Node next=null;
        public Node(int val){this.val=val;}
    }
    
    public MaxQueue() {
        this.root=new Node(-1);
        root.next=null;
        this.max=root;
        this.tail=root;
    }
    
    public int max_value() {
        if(root==tail)return -1;
        return max.val;

    }
    
    public void push_back(int value) {
        Node tmp=new Node(value);
        tail.next=tmp;
        tail=tmp;

        if(max==root||max.val<=value)max=tmp;
    }
    
    public int pop_front() {
        if(tail==root)return -1;
        
        root=root.next;//root并不代表实际节点
        if(root==max){//最大值要出去了，重新更新max
            Node head=root.next;//head-tail才代表实际的队列
            max=head;
            while(head!=null){
                max=max.val<=head.val?head:max;
                head=head.next;
            }
        }
        max=(max==null)?root:max;
        
        return root.val;
    }
}
```



#[面试题 02.04 - 分割链表](https://leetcode-cn.com/problems/partition-list-lcci/) 

>  
>  编写程序以 x 为基准分割链表，使得所有小于 x 的节点排在大于或等于 x 的节点之前。如果链表中包含 x，x 只需出现在小于 x 的元素之后(如下所示)。分割元素 x 只需处于“右半部分”即可，其不需要被置于左右两部分之间。
>
>  **示例:**
>
>  ```sql
>  输入: head = 3->5->8->5->10->2->1, x = 5
>  输出: 3->1->2->10->5->5->8
>  ```

```java
 public ListNode partition(ListNode head, int x) {
        ListNode ans = new ListNode(-1), t = ans;
        ListNode i = head, pre = null;
        for (; i != null; i = i.next) {
            if (i.val < x) {
                t.next = i;
                t = t.next;
                if (pre == null)
                    head = head.next;
                else pre.next = i.next;
                continue;
            }
            pre = i;
        }
        for (; head != null; head = head.next, t = t.next)
            t.next = head;
        return ans.next;
    }
```



#[面试题 01.02 - 判定是否互为字符重排](https://leetcode-cn.com/problems/check-permutation-lcci/) 

>  
>  给定两个字符串 `s1` 和 `s2`，请编写一个程序，确定其中一个字符串的字符重新排列后，能否变成另一个字符串。
>
>  **示例 1：**
>
>  ```
>  输入: s1 = "abc", s2 = "bca"
>  输出: true 
>  ```
>
>  **示例 2：**
>
>  ```
>  输入: s1 = "abc", s2 = "bad"
>  输出: false
>  ```
>
>  **说明：**
>
>  - `0 <= len(s1) <= 100`
>  - `0 <= len(s2) <= 100`

```java
public boolean CheckPermutation(String s1, String s2) {
      if(s1.length() != s2.length()){
            return false;
        }
        char[] c1 = s1.toCharArray();
        char[] c2 = s2.toCharArray();
        Arrays.sort(c1);
        Arrays.sort(c2);
        for(int i = 0 ; i < c1.length ; i ++){
            if(c1[i] != c2[i]){
                return false;
            }
        }
        return true;
    }
```



#[154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/) 

>  假设按照升序排序的数组在预先未知的某个点上进行了旋转。
>
>  ( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。
>
>  请找出其中最小的元素。
>
>  注意数组中可能存在重复的元素。
>
>  示例 1：
>
>  输入: [1,3,5]
>  输出: 1
>  示例 2：
>
>  输入: [2,2,2,0,1]
>  输出: 0
>  说明：
>
>  这道题是 寻找旋转排序数组中的最小值 的延伸题目。
>  允许重复会影响算法的时间复杂度吗？会如何影响，为什么？

```java
public int findMin(int[] numbers) {
        int l = 0, r = numbers.length - 1;
        while (l < r) {
            int mid = ((r - l) / 2) + l;
            //只要右边比中间大，那右边一定是有序数组
            if (numbers[r] > numbers[mid]) {
                r = mid;
            } else if (numbers[r] < numbers[mid]) {
                l = mid + 1;
             //去重
            } else r--;
        }
        return numbers[l];
    }
```



#[剑指 Offer 28 - 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/) 

>  请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。
>
>  例如，二叉树 [1,2,2,3,4,4,3] 是对称的。
>
>      1
>     / \
>    2   2
>   / \ / \
>  3  4 4  3
>  但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:
>
>      1
>     / \
>    2   2
>     \   \
>     3    3
>
>   
>
>  示例 1：
>
>  输入：root = [1,2,2,3,4,4,3]
>  输出：true
>  示例 2：
>
>  输入：root = [1,2,2,null,3,null,3]
>  输出：false
>
>
>  限制：
>
>  0 <= 节点个数 <= 1000
>
>  注意：本题与主站 101 题相同：https://leetcode-cn.com/problems/symmetric-tree/
>

```java
   public boolean isSymmetric(TreeNode root) {
        if (root == null)
            return true;
        return helper(root.left, root.right);
    }

    public boolean helper(TreeNode root1, TreeNode root2) {
        if (root1 == null && root2 == null)
            return true;
        if (root1 == null || root2 == null)
            return false;
        return root1.val == root2.val && helper(root1.left, root2.right) &&
            helper(root1.right, root2.left);
    }
```



#[剑指 Offer 55 - I - 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/) 

>  输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。
>
>  例如：
>
>  给定二叉树 [3,9,20,null,null,15,7]，
>
>      3
>     / \
>    9  20
>      /  \
>     15   7
>  返回它的最大深度 3 。

```java
 public int maxDepth(TreeNode root) {
        if(root == null) return 0;
        return Math.max(maxDepth(root.left),maxDepth(root.right))+1;
    }
```



#[68. 文本左右对齐](https://leetcode-cn.com/problems/text-justification/)

>  
>  给定一个单词数组和一个长度 *maxWidth*，重新排版单词，使其成为每行恰好有 *maxWidth* 个字符，且左右两端对齐的文本。
>
>  你应该使用“贪心算法”来放置给定的单词；也就是说，尽可能多地往每行中放置单词。必要时可用空格 `' '` 填充，使得每行恰好有 *maxWidth* 个字符。
>
>  要求尽可能均匀分配单词间的空格数量。如果某一行单词间的空格不能均匀分配，则左侧放置的空格数要多于右侧的空格数。
>
>  文本的最后一行应为左对齐，且单词之间不插入**额外的**空格。
>
>  **说明:**
>
>  - 单词是指由非空格字符组成的字符序列。
>  - 每个单词的长度大于 0，小于等于 *maxWidth*。
>  - 输入单词数组 `words` 至少包含一个单词。
>
>  **示例:**
>
>  ```
>  输入:
>  words = ["This", "is", "an", "example", "of", "text", "justification."]
>  maxWidth = 16
>  输出:
>  [
>     "This    is    an",
>     "example  of text",
>     "justification.  "
>  ]
>  ```
>
>  **示例 2:**
>
>  ```
>  输入:
>  words = ["What","must","be","acknowledgment","shall","be"]
>  maxWidth = 16
>  输出:
>  [
>    "What   must   be",
>    "acknowledgment  ",
>    "shall be        "
>  ]
>  解释: 注意最后一行的格式应为 "shall be    " 而不是 "shall     be",
>       因为最后一行应为左对齐，而不是左右两端对齐。       
>       第二行同样为左对齐，这是因为这行只包含一个单词。
>  ```
>
>  **示例 3:**
>
>  ```
>  输入:
>  words = ["Science","is","what","we","understand","well","enough","to","explain",
>           "to","a","computer.","Art","is","everything","else","we","do"]
>  maxWidth = 20
>  输出:
>  [
>    "Science  is  what we",
>    "understand      well",
>    "enough to explain to",
>    "a  computer.  Art is",
>    "everything  else  we",
>    "do                  "
>  ]
>  ```

```java
    public List<String> fullJustify(String[] words, int maxWidth) {
        List<String>list=new ArrayList();
        int arr=0,le=0,ri=0;
        while(ri<words.length){
            if(arr+words[ri].length()<maxWidth-(ri-le)){ 
                arr+=words[ri].length(); ri++;
            }
            else if(arr+words[ri].length()==maxWidth-(ri-le)){
                StringBuilder sb=new StringBuilder();
                for(int i=le;i<ri;i++)sb.append(words[i]).append(" ");
                sb.append(words[ri]);
                list.add(sb.toString());
                arr=0;ri++;le=ri;
            }
            else if(arr+words[ri].length()>maxWidth-(ri-le)){

            if(ri>le&&ri==le+1){
             StringBuilder sb=new StringBuilder();
             sb.append(words[le]);
             while(sb.length()<maxWidth)sb.append(" ");
             list.add(sb.toString()); 
             arr=0;le=ri; 
            }
                else {
                StringBuilder sb=new StringBuilder();
                int ce=(maxWidth-arr)%(ri-le-1);
                for(int i=le;i<ri-1;i++){sb.append(words[i]);     
                for(int ii=0;ii<(maxWidth-arr)/(ri-le-1);ii++)sb.append(" ");
                if(ce>0){sb.append(" ");ce--;}
                }sb.append(words[ri-1]);
                list.add(sb.toString());
                arr=0;le=ri;}
            }
           
        }
        if(ri>le){
             StringBuilder sb=new StringBuilder();
             for(int i=le;i<ri-1;i++)
             sb.append(words[i]).append(" ");
             sb.append(words[ri-1]);
             while(sb.length()<maxWidth)sb.append(" ");
             list.add(sb.toString());  
        }
        return list;
    }

```



#[168. Excel表列名称](https://leetcode-cn.com/problems/excel-sheet-column-title/) 

>  给定一个正整数，返回它在 Excel 表中相对应的列名称。
>
>  例如，
>
>      1 -> A
>      2 -> B
>      3 -> C
>      ...
>      26 -> Z
>      27 -> AA
>      28 -> AB 
>      ...
>  示例 1:
>
>  输入: 1
>  输出: "A"
>  示例 2:
>
>  输入: 28
>  输出: "AB"
>  示例 3:
>
>  输入: 701
>  输出: "ZY"

```java
 public String convertToTitle(int n) {
           if (n <= 0) {
            return "";
        }
        StringBuilder sb = new StringBuilder();
        while (n > 0) {
            n--;
            sb.append((char) (n % 26 + 'A'));
            n =n / 26;
        }
        return sb.reverse().toString();
    }
```



#[852. 山脉数组的峰顶索引](https://leetcode-cn.com/problems/peak-index-in-a-mountain-array/) 

>  我们把符合下列属性的数组 A 称作山脉：
>
>  A.length >= 3
>  存在 0 < i < A.length - 1 使得A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]
>  给定一个确定为山脉的数组，返回任何满足 A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1] 的 i 的值。
>
>   
>
>  示例 1：
>
>  输入：[0,1,0]
>  输出：1
>  示例 2：
>
>  输入：[0,2,1,0]
>  输出：1
>
>
>  提示：
>
>  3 <= A.length <= 10000
>  0 <= A[i] <= 10^6
>  A 是如上定义的山脉

```java
 public int peakIndexInMountainArray(int[] A) {
        int low = 0, high = A.length - 1;
        int mid = (low + high) / 2;
        while (mid <= high && mid > 0 && mid >= low) {
            if (A[mid] > A[mid - 1] && A[mid] > A[mid + 1])
                return mid;
            else if (A[mid] < A[mid - 1])
                high = mid - 1;
            else if (A[mid] < A[mid + 1])
                low = mid + 1;
            mid = (low + high) / 2;
        }
        return A[mid] > A[mid + 1] ? mid : mid + 1;
    }
```



#[632. 最小区间](https://leetcode-cn.com/problems/smallest-range-covering-elements-from-k-lists/) 

>  你有 k 个 非递减排列 的整数列表。找到一个 最小 区间，使得 k 个列表中的每个列表至少有一个数包含在其中。
>
>  我们定义如果 b-a < d-c 或者在 b-a == d-c 时 a < c，则区间 [a,b] 比 [c,d] 小。
>
>   
>
>  示例 1：
>
>  输入：nums = [[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]
>  输出：[20,24]
>  解释： 
>  列表 1：[4, 10, 15, 24, 26]，24 在区间 [20,24] 中。
>  列表 2：[0, 9, 12, 20]，20 在区间 [20,24] 中。
>  列表 3：[5, 18, 22, 30]，22 在区间 [20,24] 中。
>  示例 2：
>
>  输入：nums = [[1,2,3],[1,2,3],[1,2,3]]
>  输出：[1,1]
>  示例 3：
>
>  输入：nums = [[10,10],[11,11]]
>  输出：[10,11]
>  示例 4：
>
>  输入：nums = [[10],[11]]
>  输出：[10,11]
>  示例 5：
>
>  输入：nums = [[1],[2],[3],[4],[5],[6],[7]]
>  输出：[1,7]
>
>
>  提示：
>
>  nums.length == k
>  1 <= k <= 3500
>  1 <= nums[i].length <= 50
>  -105 <= nums[i][j] <= 105
>  nums[i] 按非递减顺序排列

```java
public int[] smallestRange(List<List<Integer>> nums) {
        int size = nums.size();
        Map<Integer, List<Integer>> indices = new HashMap<Integer, List<Integer>>();
        int xMin = Integer.MAX_VALUE, xMax = Integer.MIN_VALUE;
        for (int i = 0; i < size; i++) {
            for (int x : nums.get(i)) {
                List<Integer> list = indices.getOrDefault(x, new ArrayList<Integer>());
                list.add(i);
                indices.put(x, list);
                xMin = Math.min(xMin, x);
                xMax = Math.max(xMax, x);
            }
        }

        int[] freq = new int[size];
        int inside = 0;
        int left = xMin, right = xMin - 1;
        int bestLeft = xMin, bestRight = xMax;

        while (right < xMax) {
            right++;
            if (indices.containsKey(right)) {
                for (int x : indices.get(right)) {
                    freq[x]++;
                    if (freq[x] == 1) {
                        inside++;
                    }
                }
                while (inside == size) {
                    if (right - left < bestRight - bestLeft) {
                        bestLeft = left;
                        bestRight = right;
                    }
                    if (indices.containsKey(left)) {
                        for (int x: indices.get(left)) {
                            freq[x]--;
                            if (freq[x] == 0) {
                                inside--;
                            }
                        }
                    }
                    left++;
                }
            }
        }

        return new int[]{bestLeft, bestRight};
    }
```



#[922. 按奇偶排序数组 II](https://leetcode-cn.com/problems/sort-array-by-parity-ii/) 

>  给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。
>
>  对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。
>
>  你可以返回任何满足上述条件的数组作为答案。
>
>   
>
>  示例：
>
>  输入：[4,2,5,7]
>  输出：[4,5,2,7]
>  解释：[4,7,2,5]，[2,5,4,7]，[2,7,4,5] 也会被接受。
>
>
>  提示：
>
>  2 <= A.length <= 20000
>  A.length % 2 == 0
>  0 <= A[i] <= 1000

```java
 public int[] sortArrayByParityII(int[] A) {
        int[] result = new int[A.length];
        int ji = 1;
        int ou = 0;
        for (int i = 0; i < A.length; i++) {
            if (A[i] % 2 == 0) {
                result[ou] = A[i];
                ou += 2; //偶数下标移动
            } else {
                result[ji] = A[i];
                ji += 2; //奇数下标移动
            }
        }
        return result;
    }
```



#[214. 最短回文串](https://leetcode-cn.com/problems/shortest-palindrome/) 

>  给定一个字符串 s，你可以通过在字符串前面添加字符将其转换为回文串。找到并返回可以用这种方式转换的最短回文串。
>
>  示例 1：
>
>  输入：s = "aacecaaa"
>  输出："aaacecaaa"
>  示例 2：
>
>  输入：s = "abcd"
>  输出："dcbabcd"
>
>
>  提示：
>
>  0 <= s.length <= 5 * 104
>  s 仅由小写英文字母组成

```java
 public String shortestPalindrome(String s) {
        int n = s.length();
        int base = 131, mod = 1000000007;
        int left = 0, right = 0, mul = 1;
        int best = -1;
        for (int i = 0; i < n; ++i) {
            left = (int) (((long) left * base + s.charAt(i)) % mod);
            right = (int) ((right + (long) mul * s.charAt(i)) % mod);
            if (left == right) {
                best = i;
            }
            mul = (int) ((long) mul * base % mod);
        }
        String add = (best == n - 1 ? "" : s.substring(best + 1));
        StringBuffer ans = new StringBuffer(add).reverse();
        ans.append(s);
        return ans.toString();
    }
```

**kmp**

```java
public String shortestPalindrome(String s) {
        int n = s.length();
        int[] fail = new int[n];
        Arrays.fill(fail, -1);
        for (int i = 1; i < n; ++i) {
            int j = fail[i - 1];
            while (j != -1 && s.charAt(j + 1) != s.charAt(i)) {
                j = fail[j];
            }
            if (s.charAt(j + 1) == s.charAt(i)) {
                fail[i] = j + 1;
            }
        }
        int best = -1;
        for (int i = n - 1; i >= 0; --i) {
            while (best != -1 && s.charAt(best + 1) != s.charAt(i)) {
                best = fail[best];
            }
            if (s.charAt(best + 1) == s.charAt(i)) {
                ++best;
            }
        }
        String add = (best == n - 1 ? "" : s.substring(best + 1));
        StringBuffer ans = new StringBuffer(add).reverse();
        ans.append(s);
        return ans.toString();
    }
```

# [459. 重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/)

>
> 给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。
>
> **示例 1:**
>
> ```
> 输入: "abab"
> 
> 输出: True
> 
> 解释: 可由子字符串 "ab" 重复两次构成。
> ```
>
> **示例 2:**
>
> ```
> 输入: "aba"
> 
> 输出: False
> ```
>
> **示例 3:**
>
> ```
> 输入: "abcabcabcabc"
> 
> 输出: True
> 
> 解释: 可由子字符串 "abc" 重复四次构成。 (或者子字符串 "abcabc" 重复两次构成。)
> ```

```java
public boolean repeatedSubstringPattern(String s) {
        return kmp(s + s, s);
    }

    public boolean kmp(String query, String pattern) {
        int n = query.length();
        int m = pattern.length();
        int[] fail = new int[m];
        Arrays.fill(fail, -1);
        for (int i = 1; i < m; ++i) {
            int j = fail[i - 1];
            while (j != -1 && pattern.charAt(j + 1) != pattern.charAt(i)) {
                j = fail[j];
            }
            if (pattern.charAt(j + 1) == pattern.charAt(i)) {
                fail[i] = j + 1;
            }
        }
        int match = -1;
        for (int i = 1; i < n - 1; ++i) {
            while (match != -1 && pattern.charAt(match + 1) != query.charAt(i)) {
                match = fail[match];
            }
            if (pattern.charAt(match + 1) == query.charAt(i)) {
                ++match;
                if (match == m - 1) {
                    return true;
                }
            }
        }
        return false;

```



#[173. 二叉搜索树迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/) 

>  
>  实现一个二叉搜索树迭代器。你将使用二叉搜索树的根节点初始化迭代器。
>
>  调用 `next()` 将返回二叉搜索树中的下一个最小的数。
>
>   
>
>  **示例：**
>
>  **![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/25/bst-tree.png)**
>
>  ```
>  BSTIterator iterator = new BSTIterator(root);
>  iterator.next();    // 返回 3
>  iterator.next();    // 返回 7
>  iterator.hasNext(); // 返回 true
>  iterator.next();    // 返回 9
>  iterator.hasNext(); // 返回 true
>  iterator.next();    // 返回 15
>  iterator.hasNext(); // 返回 true
>  iterator.next();    // 返回 20
>  iterator.hasNext(); // 返回 false
>  ```
>
>   
>
>  **提示：**
>
>  - `next()` 和 `hasNext()` 操作的时间复杂度是 O(1)，并使用 O(*h*) 内存，其中 *h* 是树的高度。
>  - 你可以假设 `next()` 调用总是有效的，也就是说，当调用 `next()` 时，BST 中至少存在一个下一个最小的数。

```java
  LinkedList<TreeNode> stack = new LinkedList<>();
    public BSTIterator(TreeNode root) {
        if(root!=null){
            left_In(root);
        }
    }
    private void left_In(TreeNode root){
        while(root!=null){
            stack.push(root);
            root = root.left;
        }
    }
    
    /** @return the next smallest number */
    public int next() {
        TreeNode tem = stack.pop();
        int ret = tem.val;
        left_In(tem.right);
        return ret;
    }
    
    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        return !stack.isEmpty();
    }
```



#[218. 天际线问题](https://leetcode-cn.com/problems/the-skyline-problem/) 

>  城市的天际线是从远处观看该城市中所有建筑物形成的轮廓的外部轮廓。现在，假设您获得了城市风光照片（图A）上显示的所有建筑物的位置和高度，请编写一个程序以输出由这些建筑物形成的天际线（图B）。
>
>  ![Buildings](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/skyline1.png)
>
>   ![Skyline Contour](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/skyline2.png)
>
>  每个建筑物的几何信息用三元组 [Li，Ri，Hi] 表示，其中 Li 和 Ri 分别是第 i 座建筑物左右边缘的 x 坐标，Hi 是其高度。可以保证 0 ≤ Li, Ri ≤ INT_MAX, 0 < Hi ≤ INT_MAX 和 Ri - Li > 0。您可以假设所有建筑物都是在绝对平坦且高度为 0 的表面上的完美矩形。
>
>  例如，图A中所有建筑物的尺寸记录为：[ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ] 。
>
>  输出是以 [ [x1,y1], [x2, y2], [x3, y3], ... ] 格式的“关键点”（图B中的红点）的列表，它们唯一地定义了天际线。关键点是水平线段的左端点。请注意，最右侧建筑物的最后一个关键点仅用于标记天际线的终点，并始终为零高度。此外，任何两个相邻建筑物之间的地面都应被视为天际线轮廓的一部分。
>
>  例如，图B中的天际线应该表示为：[ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ]。
>
>  说明:
>
>  任何输入列表中的建筑物数量保证在 [0, 10000] 范围内。
>  输入列表已经按左 x 坐标 Li  进行升序排列。
>  输出列表必须按 x 位排序。
>  输出天际线中不得有连续的相同高度的水平线。例如 [...[2 3], [4 5], [7 5], [11 5], [12 7]...] 是不正确的答案；三条高度为 5 的线应该在最终输出中合并为一个：[...[2 3], [4 5], [12 7], ...]

```java
// 线段树 来源 算法无遗策 公众号
    public List<List<Integer>> getSkyline(int[][] buildings) {
        int len = buildings.length;
        if (len == 0) return new ArrayList<>();
        return segment(buildings, 0, len - 1);
    }

    private List<List<Integer>> segment(int[][] buildings, int l, int r) {
        // 创建返回值
        List<List<Integer>> res = new ArrayList<>();

        // 找到树底下的结束条件 -> 一个建筑物
        if (l == r) {
            res.add(Arrays.asList(buildings[l][0], buildings[l][2])); // 左上端坐标
            res.add(Arrays.asList(buildings[l][1], 0)); // 右下端坐标
            return res;
        }

        int mid = l + (r - l) / 2; // 取中间值

        // 左边递归
        List<List<Integer>> left = segment(buildings, l, mid);

        // 右边递归
        List<List<Integer>> right = segment(buildings, mid + 1, r);

        // 左右合并

        // 创建left 和 right 的索引位置
        int m = 0, n = 0;
        // 创建left 和 right 目前的高度
        int lpreH = 0, rpreH = 0;
        // 两个坐标
        int leftX, leftY, rightX, rightY;
        while (m < left.size() || n < right.size()) {

            // 当有一边完全加入到res时，则加入剩余的那部分
            if (m >= left.size()) res.add(right.get(n++));
            else if (n >= right.size()) res.add(left.get(m++));

            else { // 开始判断left 和 right
                leftX = left.get(m).get(0); // 不会出现null，可以直接用int类型
                leftY = left.get(m).get(1);
                rightX = right.get(n).get(0);
                rightY = right.get(n).get(1);

                if (leftX < rightX) {
                   if (leftY > rpreH) res.add(left.get(m));
                   else if (lpreH > rpreH) res.add(Arrays.asList(leftX, rpreH));
                    lpreH = leftY;
                    m++;
                } else if (leftX > rightX) {
                   if (rightY > lpreH) res.add(right.get(n));
                   else if (rpreH > lpreH) res.add(Arrays.asList(rightX, lpreH));
                    rpreH = rightY;
                    n++;
                } else { // left 和 right 的横坐标相等
                  if (leftY >= rightY && leftY != (lpreH > rpreH ? lpreH : rpreH)) res.add(left.get(m));
                    else if (leftY <= rightY && rightY != (lpreH > rpreH ? lpreH : rpreH)) res.add(right.get(n));
                    lpreH = leftY;
                    rpreH = rightY;
                    m++;
                    n++;
                }
            }
        }
        return res;
    }
```



#[278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/) 

>  
>  你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。
>
>  假设你有 `n` 个版本 `[1, 2, ..., n]`，你想找出导致之后所有版本出错的第一个错误的版本。
>
>  你可以通过调用 `bool isBadVersion(version)` 接口来判断版本号 `version` 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。
>
>  **示例:**
>
>  ```java
>  给定 n = 5，并且 version = 4 是第一个错误的版本。
>  
>  调用 isBadVersion(3) -> false
>  调用 isBadVersion(5) -> true
>  调用 isBadVersion(4) -> true
>  
>  所以，4 是第一个错误的版本。 
>  ```

```java
  public int firstBadVersion(int n) {
        if (n <= 0) {
            return 0;
        }

        if (isBadVersion(1)) {
            return 1;
        } else if (!isBadVersion(n)) {
            return Integer.MAX_VALUE;
        }
        
        int left = 1;
        int right = n;
        
        int mid;
        while (true) {
            mid = left + (right - left) / 2;
            if (mid == left) {
                return right;
            }
            if (isBadVersion(mid)) {
                right = mid;
            } else {
                left = mid;
            }
        }
    }
```



#[698. 划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/) 

>  
>  给定一个整数数组 `nums` 和一个正整数 `k`，找出是否有可能把这个数组分成 `k` 个非空子集，其总和都相等。
>
>  **示例 1：**
>
>  ```
>  输入： nums = [4, 3, 2, 3, 5, 2, 1], k = 4
>  输出： True
>  说明： 有可能将其分成 4 个子集（5），（1,4），（2,3），（2,3）等于总和。
>  ```
>
>   
>
>  **提示：**
>
>  - `1 <= k <= len(nums) <= 16`
>  - `0 < nums[i] < 10000`

```java
enum Result { TRUE, FALSE }		
boolean search(int used, int todo, Result[] memo, int[] nums, int target) {
        if (memo[used] == null) {
            memo[used] = Result.FALSE;
            int targ = (todo - 1) % target + 1;
            for (int i = 0; i < nums.length; i++) {
                if ((((used >> i) & 1) == 0) && nums[i] <= targ) {
                    if (search(used | (1<<i), todo - nums[i], memo, nums, target)) {
                        memo[used] = Result.TRUE;
                        break;
                    }
                }
            }
        }
        return memo[used] == Result.TRUE;
    }
    public boolean canPartitionKSubsets(int[] nums, int k) {
        int sum = Arrays.stream(nums).sum();
        if (sum % k > 0) return false;

        Result[] memo = new Result[1 << nums.length];
        memo[(1 << nums.length) - 1] = Result.TRUE;
        return search(0, sum, memo, nums, sum / k);
    }
```



#[870. 优势洗牌](https://leetcode-cn.com/problems/advantage-shuffle/) 

>  给定两个大小相等的数组 A 和 B，A 相对于 B 的优势可以用满足 A[i] > B[i] 的索引 i 的数目来描述。
>
>  返回 A 的任意排列，使其相对于 B 的优势最大化。
>
>   
>
>  示例 1：
>
>  输入：A = [2,7,11,15], B = [1,10,4,11]
>  输出：[2,11,7,15]
>  示例 2：
>
>  输入：A = [12,24,8,32], B = [13,25,32,11]
>  输出：[24,32,8,12]
>
>
>  提示：
>
>  1 <= A.length = B.length <= 10000
>  0 <= A[i] <= 10^9
>  0 <= B[i] <= 10^9

```java
  public int[] advantageCount(int[] A, int[] B) {
        Arrays.sort(A);
        List<Integer> list = new ArrayList<>();
        for (Integer i : A) {
            list.add(i);
        }
        for (int i = 0; i < B.length; i++) {
            A[i] = findBigMin(list, B[i]);
        }
        for (int i = 0; i < A.length; i++) {
            if (A[i] == -1) {
                A[i] = list.get(0);
                list.remove(0);
            }
        }
        return A;
    }

    private int findBigMin(List<Integer> list, int x) {
        for (Integer integer : list) {
            if (integer > x) {
                list.remove((Integer) integer);
                return integer;
            }
        }
        return -1;
    }
```



#[73. 矩阵置零](https://leetcode-cn.com/problems/set-matrix-zeroes/) 

>  给定一个 m x n 的矩阵，如果一个元素为 0，则将其所在行和列的所有元素都设为 0。请使用原地算法。
>
>  示例 1:
>
>  输入: 
>  [
>    [1,1,1],
>    [1,0,1],
>    [1,1,1]
>  ]
>  输出: 
>  [
>    [1,0,1],
>    [0,0,0],
>    [1,0,1]
>  ]
>  示例 2:
>
>  输入: 
>  [
>    [0,1,2,0],
>    [3,4,5,2],
>    [1,3,1,5]
>  ]
>  输出: 
>  [
>    [0,0,0,0],
>    [0,4,5,0],
>    [0,3,1,0]
>  ]
>  进阶:
>
>  一个直接的解决方案是使用  O(mn) 的额外空间，但这并不是一个好的解决方案。
>  一个简单的改进方案是使用 O(m + n) 的额外空间，但这仍然不是最好的解决方案。
>  你能想出一个常数空间的解决方案吗？

```java
 public void setZeroes(int[][] matrix) {
                Boolean isCol = false;
        int R = matrix.length;
        int C = matrix[0].length;

        for (int i = 0; i < R; i++) {

            // Since first cell for both first row and first column is the same i.e. matrix[0][0]
            // We can use an additional variable for either the first row/column.
            // For this solution we are using an additional variable for the first column
            // and using matrix[0][0] for the first row.
            if (matrix[i][0] == 0) {
                isCol = true;
            }

            for (int j = 1; j < C; j++) {
                // If an element is zero, we set the first element of the corresponding row and column to 0
                if (matrix[i][j] == 0) {
                    matrix[0][j] = 0;
                    matrix[i][0] = 0;
                }
            }
        }

        // Iterate over the array once again and using the first row and first column, update the elements.
        for (int i = 1; i < R; i++) {
            for (int j = 1; j < C; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }

        // See if the first row needs to be set to zero as well
        if (matrix[0][0] == 0) {
            for (int j = 0; j < C; j++) {
                matrix[0][j] = 0;
            }
        }

        // See if the first column needs to be set to zero as well
        if (isCol) {
            for (int i = 0; i < R; i++) {
                matrix[i][0] = 0;
            }
        }
    }
```



#[1116. 打印零与奇偶数](https://leetcode-cn.com/problems/print-zero-even-odd/) 

>  假设有这么一个类：
>
>  class ZeroEvenOdd {
>    public ZeroEvenOdd(int n) { ... }      // 构造函数
>    public void zero(printNumber) { ... }  // 仅打印出 0
>    public void even(printNumber) { ... }  // 仅打印出 偶数
>    public void odd(printNumber) { ... }   // 仅打印出 奇数
>  }
>  相同的一个 ZeroEvenOdd 类实例将会传递给三个不同的线程：
>
>  线程 A 将调用 zero()，它只输出 0 。
>  线程 B 将调用 even()，它只输出偶数。
>  线程 C 将调用 odd()，它只输出奇数。
>  每个线程都有一个 printNumber 方法来输出一个整数。请修改给出的代码以输出整数序列 010203040506... ，其中序列的长度必须为 2n。
>
>   
>
>  示例 1：
>
>  输入：n = 2
>  输出："0102"
>  说明：三条线程异步执行，其中一个调用 zero()，另一个线程调用 even()，最后一个线程调用odd()。正确的输出为 "0102"。
>  示例 2：
>
>  输入：n = 5
>  输出："0102030405"

```java
   private int n;
    private Lock lock;
    private int flag;
    private Condition zero;
    private Condition even;
    private Condition odd;

    public ZeroEvenOdd(int n) {
        this.n = n;
        this.flag = 1;
        this.lock = new ReentrantLock();
        this.zero = lock.newCondition();
        this.even = lock.newCondition();
        this.odd = lock.newCondition();
    }

    // printNumber.accept(x) outputs "x", where x is an integer.
    public void zero(IntConsumer printNumber) throws InterruptedException {
        for (int i = 1;i <= n;i++) {
            try {
                lock.lock();
                while (flag != 1) zero.await();
                printNumber.accept(0);
                if (i % 2 == 0) {
                    flag = 2;
                    even.signal();
                }else {
                    flag = 3;
                    odd.signal();
                }
            }finally {
                lock.unlock();
            }
        }
    }

    public void even(IntConsumer printNumber) throws InterruptedException {
        for (int i = 2;i <= n;i += 2) {
            try {
                lock.lock();
                while (flag != 2) even.await();
                printNumber.accept(i);
                flag = 1;
                zero.signal();
            }finally {
                lock.unlock();
            }
        }
    }

    public void odd(IntConsumer printNumber) throws InterruptedException {
        for (int i = 1;i <= n;i += 2) {
            try {
                lock.lock();
                while (flag != 3) odd.await();
                printNumber.accept(i);
                flag = 1;
                zero.signal();
            }finally {
                lock.unlock();
            }
        }
    }
```

