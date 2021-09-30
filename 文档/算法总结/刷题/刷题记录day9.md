[toc]

#[181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/) 

>   Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。
>
>   +----+-------+--------+-----------+
>   | Id | Name  | Salary | ManagerId |
>   +----+-------+--------+-----------+
>   | 1  | Joe   | 70000  | 3         |
>   | 2  | Henry | 80000  | 4         |
>   | 3  | Sam   | 60000  | NULL      |
>   | 4  | Max   | 90000  | NULL      |
>   +----+-------+--------+-----------+
>   给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。
>
>   +----------+
>   | Employee |
>   +----------+
>   | Joe      |
>   +----------+

```java
select 
    e.Name Employee
from Employee e left join (select distinct Id,Salary from Employee) m  
    on e.ManagerId  = m.Id 
where e.Salary  > m.Salary 
```

#[328. 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/)

>   给定一个单链表，把所有的奇数节点和偶数节点分别排在一起。请注意，这里的奇数节点和偶数节点指的是节点编号的奇偶性，而不是节点的值的奇偶性。
>
>   请尝试使用原地算法完成。你的算法的空间复杂度应为 O(1)，时间复杂度应为 O(nodes)，nodes 为节点总数。
>
>   示例 1:
>
>   输入: 1->2->3->4->5->NULL
>   输出: 1->3->5->2->4->NULL
>   示例 2:
>
>   输入: 2->1->3->5->6->4->7->NULL 
>   输出: 2->3->6->7->1->5->4->NULL
>   说明:
>
>   应当保持奇数节点和偶数节点的相对顺序。
>   链表的第一个节点视为奇数节点，第二个节点视为偶数节点，以此类推。

```java
public ListNode oddEvenList(ListNode head) {
        if (head == null) return head;
        ListNode odd = head, even = head.next, evenhead = even;
        while (odd.next != null && even.next != null) {
            odd.next = odd.next.next;
            even.next = even.next.next;
            odd = odd.next;
            even = even.next;
        }
        odd.next = evenhead;
        return head;
    }
```

# [59. 螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii/)

>   给定一个正整数 n，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。
>
>   示例:
>
>   输入: 3
>   输出:
>   [
>    [ 1, 2, 3 ],
>    [ 8, 9, 4 ],
>    [ 7, 6, 5 ]
>   ]

```java
    public int[][] generateMatrix(int n) {
              int[][] arr = new int[n][n];
        int c = 1, j = 0;
        while (c <= n * n) {
        
            for (int i = j; i < n - j; i++)
                arr[j][i] = c++;
            for (int i = j + 1; i < n - j; i++)
                arr[i][n - j - 1] = c++;
            for (int i = n - j - 2; i >= j; i--)
                arr[n - j - 1][i] = c++;
            for (int i = n -j - 2; i > j; i--)
                arr[i][j] = c++;

            j++;
        }

        return arr;
    }
```

# [407. 接雨水 II](https://leetcode-cn.com/problems/trapping-rain-water-ii/)

>   
>   给你一个 `m x n` 的矩阵，其中的值均为非负整数，代表二维高度图每个单元的高度，请计算图中形状最多能接多少体积的雨水。
>
>    ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/rainwater_empty.png)
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/rainwater_fill.png)
>
>   **示例：**
>
>   ```
>   给出如下 3x6 的高度图:
>   [
>     [1,4,3,1,3,2],
>     [3,2,1,3,2,4],
>     [2,3,3,2,3,1]
>   ]
>   
>   返回 4 。
>   ```

```java
  public int trapRainWater(int[][] heightMap) {
    if (heightMap == null || heightMap.length <= 2 || heightMap[0].length <= 2)
        return 0;
    PriorityQueue<Cell> queue = new PriorityQueue<>(Comparator.comparingInt((Cell cell) -> cell.height));
    int m = heightMap.length;
    int n = heightMap[0].length;
    boolean[][] visited = new boolean[m][n];
    for (int i = 0; i < m; i++) {
        visited[i][0] = visited[i][n-1] = true;
        queue.add(new Cell(i, 0, heightMap[i][0]));
        queue.add(new Cell(i, n-1, heightMap[i][n-1]));
    }
    for (int i = 1; i < n-1; i++) {
        visited[0][i] = visited[m-1][i] = true;
        queue.add(new Cell(0, i, heightMap[0][i]));
        queue.add(new Cell(m-1, i, heightMap[m-1][i]));
    }
    int result = 0;
    int[][] bounds = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    while (!queue.isEmpty()) {
        Cell cell = queue.poll();
        for (int[] bound : bounds) {
            int row = cell.row + bound[0];
            int col = cell.col + bound[1];
            if (row >= 0 && row < m && col >= 0 && col < n && !visited[row][col]) {
                result += Math.max(0, cell.height - heightMap[row][col]);
                visited[row][col] = true;
                queue.add(new Cell(row, col, Math.max(cell.height, heightMap[row][col])));
            }
        }
    }
    return result;
}

private static class Cell {
    private int row;
    private int col;
    private int height;

    public Cell(int row, int col, int height) {
        this.row = row;
        this.col = col;
        this.height = height;
    }
}
```

# [912. 排序数组](https://leetcode-cn.com/problems/sort-an-array/)

>   单独开一个文档

```java

```

# [剑指 Offer 63 - 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

>   假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？
>
>    
>
>   示例 1:
>
>   输入: [7,1,5,3,6,4]
>   输出: 5
>   解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
>        注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
>   示例 2:
>
>   输入: [7,6,4,3,1]
>   输出: 0
>   解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
>
>
>   限制：
>
>   0 <= 数组长度 <= 10^5
>

```java
  public int maxProfit(int[] prices) {
        int buy = Integer.MAX_VALUE;
        int res = 0;
        for(int num : prices){
            buy = Math.min(num,buy);
            res = Math.max(res,num - buy);
        }
        return res;
    }
```

# [剑指 Offer 25 - 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

>   输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。
>
>   示例1：
>
>   输入：1->2->4, 1->3->4
>   输出：1->1->2->3->4->4
>   限制：
>
>   0 <= 链表长度 <= 1000
>

```java
  public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode res = new ListNode(-1);
        ListNode p = l1,q = l2,cur = res;
        while(p != null && q !=null){
            if(p.val < q.val){
                cur.next = p;
                p = p.next;
            }else{
                cur.next = q;
                q = q.next;
            }
            cur = cur.next;
        }
        cur.next = p == null ? q : p;
        return res.next;
    }
```

# [18. 四数之和](https://leetcode-cn.com/problems/4sum/)

>   给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。
>
>   注意：
>
>   答案中不可以包含重复的四元组。
>
>   示例：
>
>   给定数组 nums = [1, 0, -1, 0, -2, 2]，和 target = 0。
>
>   满足要求的四元组集合为：
>   [
>     [-1,  0, 0, 1],
>     [-2, -1, 1, 2],
>     [-2,  0, 0, 2]
>   ]

```java
  public List<List<Integer>> fourSum(int[] num, int target) {
        List<List<Integer>> res = new ArrayList<>();
        if (num == null || num.length < 4) {
            return res;
        }
        Arrays.sort(num);
        for (int i = 0; i < num.length - 3; i++) {
            //去除重复
            if (i > 0 && num[i - 1] == num[i]) continue;
            for (int j = i + 1; j < num.length - 2; j++) {
                 if (j != i + 1 && num[j] == num[j - 1]) continue;
                int k = j + 1;
                int m = num.length - 1;
                while (k < m) {
                    int value = num[i] + num[j] + num[k] + num[m];
                    if (value == target) {
                        //处理重复
                        res.add(Arrays.asList(num[i], num[j], num[k], num[m]));
                        while (k < m && num[k] == num[k + 1]) k++;
                        while (k < m && num[m] == num[m - 1]) m--;
                        k++;
                        m--;
                    } else if (value < target) {
                        k++;
                    } else {
                        m--;
                    }
                }
            }
        }
        return res;
    }
```

#[232. 用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

>   请你仅使用两个栈实现先入先出队列。队列应当支持一般队列的支持的所有操作（push、pop、peek、empty）：
>
>   实现 MyQueue 类：
>
>   void push(int x) 将元素 x 推到队列的末尾
>   int pop() 从队列的开头移除并返回元素
>   int peek() 返回队列开头的元素
>   boolean empty() 如果队列为空，返回 true ；否则，返回 false
>
>
>   说明：
>
>   你只能使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
>   你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
>
>
>   进阶：
>
>   你能否实现每个操作均摊时间复杂度为 O(1) 的队列？换句话说，执行 n 个操作的总时间复杂度为 O(n) ，即使其中一个操作可能花费较长时间。
>
>
>   示例：
>
>   输入：
>   ["MyQueue", "push", "push", "peek", "pop", "empty"]
>   [[], [1], [2], [], [], []]
>   输出：
>   [null, null, null, 1, 1, false]
>
>   解释：
>   MyQueue myQueue = new MyQueue();
>   myQueue.push(1); // queue is: [1]
>   myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
>   myQueue.peek(); // return 1
>   myQueue.pop(); // return 1, queue is [2]
>   myQueue.empty(); // return false

```java
class MyQueue {

    Stack<Integer> s1;
    Stack<Integer> s2;

    public MyQueue() {
        s1 = new Stack();
        s2 = new Stack();
    }

    /**
     * Push element x to the back of queue.
     */
    public void push(int x) {
        s1.push(x);
    }

    /**
     * Removes the element from in front of queue and returns that element.
     */
    public int pop() {
        int val = peek();
        s2.pop();
        return val;
    }

    /**
     * Get the front element.
     */
    public int peek() {
        if (s2.isEmpty()) {
            while (!s1.isEmpty()) {
                s2.push(s1.pop());
            }
        }
        return s2.peek();
    }

    /**
     * Returns whether the queue is empty.
     */
    public boolean empty() {
        return s1.size() + s2.size() == 0;
    }
}
```

# [557. 反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/)

>   给定一个字符串，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。
>
>    
>
>   示例：
>
>   输入："Let's take LeetCode contest"
>   输出："s'teL ekat edoCteeL tsetnoc"
>
>
>   提示：
>
>   在字符串中，每个单词由单个空格分隔，并且字符串中不会有任何额外的空格。
>

```java
 public String reverseWords(String s) {
        if(s == null || s.length() == 0) return s;
        StringBuilder re = new StringBuilder();
        String[] strings = s.split(" ");
        for (String str : strings) {
            StringBuilder sb = new StringBuilder(str);
            re.append(sb.reverse().toString());
            //多拼了一个空格,最后要substring
            re.append(" ");
        }
         return re.toString().substring(0, re.toString().length() - 1);
    }
```

# [43. 字符串相乘](https://leetcode-cn.com/problems/multiply-strings/)

>   给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。
>
>   示例 1:
>
>   输入: num1 = "2", num2 = "3"
>   输出: "6"
>   示例 2:
>
>   输入: num1 = "123", num2 = "456"
>   输出: "56088"
>   说明：
>
>   num1 和 num2 的长度小于110。
>   num1 和 num2 只包含数字 0-9。
>   num1 和 num2 均不以零开头，除非是数字 0 本身。
>   不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。

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

#[75. 颜色分类](https://leetcode-cn.com/problems/sort-colors/)

>   给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。
>
>   此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。
>
>    
>
>   进阶：
>
>   你可以不使用代码库中的排序函数来解决这道题吗？
>   你能想出一个仅使用常数空间的一趟扫描算法吗？
>
>
>   示例 1：
>
>   输入：nums = [2,0,2,1,1,0]
>   输出：[0,0,1,1,2,2]
>   示例 2：
>
>   输入：nums = [2,0,1]
>   输出：[0,1,2]
>   示例 3：
>
>   输入：nums = [0]
>   输出：[0]
>   示例 4：
>
>   输入：nums = [1]
>   输出：[1]

```java
    //颜色分类
    public static void sortColors(int[] nums) {
        //双指针
        int low = 0, high = nums.length - 1;
        int i = 0;
        while (i <= high) {
            if (nums[i] == 0) {
                swap(nums, low, i);
                ++low;
                ++i;
            } else if (nums[i] == 1) {
                ++i;
            } else if (nums[i] == 2) {
                swap(nums, high, i);
                --high;
            }
        }
    }

    private static void swap(int[] nums, int low, int i) {
        int tmp = nums[i];
        nums[i] = nums[low];
        nums[low] = tmp;
    }
```

# [213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

>   你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
>
>   给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，能够偷窃到的最高金额。
>
>    
>
>   示例 1：
>
>   输入：nums = [2,3,2]
>   输出：3
>   解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。
>   示例 2：
>
>   输入：nums = [1,2,3,1]
>   输出：4
>   解释：你可以先偷窃 1 号房屋（金额 = 1），然后偷窃 3 号房屋（金额 = 3）。
>        偷窃到的最高金额 = 1 + 3 = 4 。
>   示例 3：
>
>   输入：nums = [0]
>   输出：0

```java
  public int rob(int[] nums) {
        	if(nums==null||nums.length==0)
    		return 0;
    	if(nums.length==1)
    		return nums[0];
    	int[] dp1 = new int[nums.length];
    	int[] dp2 = new int[nums.length];
    	dp1[1] = nums[0]; //从第1个房屋开始偷
    	dp2[1] = nums[1]; //从第2个房屋开始偷
    	for(int i=2;i<nums.length;i++) {
    		dp1[i] = Math.max( dp1[i-2]+nums[i-1],dp1[i-1] );
    		dp2[i] = Math.max( dp2[i-2]+nums[i], dp2[i-1]);
    	}
    	return Math.max( dp1[nums.length-1], dp2[nums.length-1] );
    }
```

#[剑指 Offer 40 - 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

>   输入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。
>
>    
>
>   示例 1：
>
>   输入：arr = [3,2,1], k = 2
>   输出：[1,2] 或者 [2,1]
>   示例 2：
>
>   输入：arr = [0,1,2,1], k = 1
>   输出：[0]
>
>
>   限制：
>
>   0 <= k <= arr.length <= 10000
>   0 <= arr[i] <= 10000

```java
    public int[] getLeastNumbers(int[] arr, int k) {
        int low=0,high=arr.length-1;
        while(low<high){
            int i = partition(arr,low,high);
            if(i>=k) high=i-1;
            if(i<=k) low=i+1;
        }
        return Arrays.copyOf(arr,k);
    }
    
    int partition(int[] nums,int low,int high){
        int pivot = nums[low];
        while(low<high){
            while(low<high&&nums[high]>=pivot)
                high--;
            nums[low]=nums[high];
            while(low<high&&nums[low]<=pivot)
                low++;
            nums[high]=nums[low];
        }
        nums[low]=pivot;
        return low;
    }
```

#[剑指 Offer 62 - 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

>   
>   0,1,,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。
>
>   例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。
>
>    
>
>   **示例 1：**
>
>   ```
>   输入: n = 5, m = 3
>   输出: 3
>   ```
>
>   **示例 2：**
>
>   ```
>   输入: n = 10, m = 17
>   输出: 2
>   ```
>
>    
>
>   **限制：**
>
>   - `1 <= n <= 10^5`
>   - `1 <= m <= 10^6`

```java
public int lastRemaining(int n, int m) {
        int ans = 0;
        // 最后一轮剩下2个人，所以从2开始反推
        for (int i = 2; i <= n; i++) {
            ans = (ans + m) % i;
        }
        return ans;
    }
```

# [剑指 Offer 36 - 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

>   输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。
>
>    
>
>   为了让您更好地理解问题，以下面的二叉搜索树为例：
>
>    ![img](https://assets.leetcode.com/uploads/2018/10/12/bstdlloriginalbst.png)
>
>   我们希望将这个二叉搜索树转化为双向循环链表。链表中的每个节点都有一个前驱和后继指针。对于双向循环链表，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。
>
>   下图展示了上面的二叉搜索树转化成的链表。“head” 表示指向链表中有最小元素的节点。
>
>    
>
>   ![img](https://assets.leetcode.com/uploads/2018/10/12/bstdllreturndll.png)
>
>    
>
>   特别地，我们希望可以就地完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中的第一个节点的指针。
>

```java
private Node pre,head;
    public Node treeToDoublyList(Node root) {
        if(root == null){
            return null;
        }
        dfs(root);
        head.left = pre;
        pre.right = head;
        return head;
    }
    private void dfs(Node root){
        if(root == null) return;
        dfs(root.left);
        if(pre  == null){
            pre = root;
            head = root;
        }else{
            pre.right = root;
            root.left = pre;
            pre = pre.right;
        }
        dfs(root.right);
    }
```

# [12. 整数转罗马数字](https://leetcode-cn.com/problems/integer-to-roman/)

>   罗马数字包含以下七种字符： I， V， X， L，C，D 和 M。
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
>   给定一个整数，将其转为罗马数字。输入确保在 1 到 3999 的范围内。

```java
 public String intToRoman(int num) {
        //900,400,40要维护一下
        int[] numbers = new int[]{1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
        String[] dic = new String[]{"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};
        StringBuilder sb =  new StringBuilder();
        for(int i = 0;i<numbers.length;i++){
            if(num >= numbers[i]){
                int count = num / numbers[i];
                for(int j = 0;j < count;j++){
                    sb.append(dic[i]);
                    num = num - numbers[i];
                }
            }
        }
        return sb.toString();
    }
```

# [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

>   给定一个整数数组 prices ，它的第 i 个元素 prices[i] 是一支给定的股票在第 i 天的价格。
>
>   设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。
>
>   注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>    
>
>   示例 1：
>
>   输入：k = 2, prices = [2,4,1]
>   输出：2
>   解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
>   示例 2：
>
>   输入：k = 2, prices = [3,2,6,5,0,3]
>   输出：7
>   解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
>        随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
>
>
>   提示：
>
>   0 <= k <= 109
>   0 <= prices.length <= 1000
>   0 <= prices[i] <= 1000

```java
 public int maxProfit(int k, int[] prices) {
        /**
        当k大于等于数组长度一半时, 问题退化为贪心问题此时采用 买卖股票的最佳时机 II
        的贪心方法解决可以大幅提升时间性能, 对于其他的k, 可以采用 买卖股票的最佳时机 III
        的方法来解决, 在III中定义了两次买入和卖出时最大收益的变量, 在这里就是k租这样的
        变量, 即问题IV是对问题III的推广, t[i][0]和t[i][1]分别表示第i比交易买入和卖出时
        各自的最大收益
        **/
        if(k < 1) return 0;
        if(k >= prices.length/2) return greedy(prices);
        int[][] t = new int[k][2];
        for(int i = 0; i < k; ++i)
            t[i][0] = Integer.MIN_VALUE;
        for(int p : prices) {
            t[0][0] = Math.max(t[0][0], -p);
            t[0][1] = Math.max(t[0][1], t[0][0] + p);
            for(int i = 1; i < k; ++i) {
                t[i][0] = Math.max(t[i][0], t[i-1][1] - p);
                t[i][1] = Math.max(t[i][1], t[i][0] + p);
            }
        }
        return t[k-1][1];
    }
    
    private int greedy(int[] prices) {
        int max = 0;
        for(int i = 1; i < prices.length; ++i) {
            if(prices[i] > prices[i-1])
                max += prices[i] - prices[i-1];
        }
        return max;
    }
```

# [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

>   给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。
>
>   示例 1:
>
>   输入: n = 12
>   输出: 3 
>   解释: 12 = 4 + 4 + 4.
>   示例 2:
>
>   输入: n = 13
>   输出: 2
>   解释: 13 = 4 + 9.

```java
  public int numSquares(int n) {
        int[] dp = new int[n+1];
        for(int i = 1; i <= n; i++){
            dp[i] = Integer.MAX_VALUE;
        }
        for(int i = 1; i <= n; i++){
            for(int j = 1; j*j <= i; j++){
                if(i >= j*j){
                    dp[i] = Math.min(dp[i],dp[i-j*j]+1);
                }
            }
        }
        return dp[n];
    }
```

#[344. 反转字符串](https://leetcode-cn.com/problems/reverse-string/)

>   编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char[] 的形式给出。
>
>   不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。
>
>   你可以假设数组中的所有字符都是 ASCII 码表中的可打印字符。
>
>    
>
>   示例 1：
>
>   输入：["h","e","l","l","o"]
>   输出：["o","l","l","e","h"]
>   示例 2：
>
>   输入：["H","a","n","n","a","h"]
>   输出：["h","a","n","n","a","H"]

```java
    public String reverseString(String s) {
          if (s == null||s=="") {
            return null;
        }
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length/2; i++) {
            char t =chars[i];
            chars[i] = chars[chars.length - 1 -i];
            chars[chars.length-1-i]= t;
        }
        return String.valueOf(chars);
    }
```

