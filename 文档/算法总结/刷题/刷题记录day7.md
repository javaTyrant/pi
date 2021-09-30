[toc]

#[62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

>   一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
>
>   机器人每次只能**向下**或者**向右**移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
>
>   问总共有多少条不同的路径？
>
>   ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/robot_maze.png)
>

```java
  public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];        
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 || j == 0)
                    dp[i][j] = 1;
                else {
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
                }
            }
        }
        return dp[m - 1][n - 1];   
    }
```

# [110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

>   给定一个二叉树，判断它是否是高度平衡的二叉树。
>
>   本题中，一棵高度平衡二叉树定义为：
>
>   > 一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1 

```java
 public static boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }
        int leftL = getHeight(root.left);
        int rightL = getHeight(root.right);
        if (Math.abs(leftL - rightL) <= 1) {
            return isBalanced(root.left) && isBalanced(root.right);
        }
        return false;
    }

    public static int getHeight(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftL = getHeight(root.left);
        int rightL = getHeight(root.right);
        return leftL > rightL ? leftL + 1 : rightL + 1;
    }
```

# [162. 寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

>   峰值元素是指其值大于左右相邻值的元素。
>
>   给定一个输入数组 nums，其中 nums[i] ≠ nums[i+1]，找到峰值元素并返回其索引。
>
>   数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。
>
>   你可以假设 nums[-1] = nums[n] = -∞。
>
>   示例 1:
>
>   输入: nums = [1,2,3,1]
>   输出: 2
>   解释: 3 是峰值元素，你的函数应该返回其索引 2。
>   示例 2:
>
>   输入: nums = [1,2,1,3,5,6,4]
>   输出: 1 或 5 
>   解释: 你的函数可以返回索引 1，其峰值元素为 2；
>        或者返回索引 5， 其峰值元素为 6。

```java
    public int findPeakElement(int[] nums) {
         int low = 0;
        int high = nums.length - 1;
        while (low < high)  //当两者相等时，
        {
            int mid = (low + high) / 2;
            if (nums[mid] < nums[mid + 1])
                low = mid + 1;  //注意为什么要+1，因为mid位置一定不是峰，而low=mid+1有可能是峰
            else
                high = mid;      //注意这里为什么不加+，因为high=mid有可能是峰
        }
        return low;
    }
```

# [560. 和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

>   
>   给定一个整数数组和一个整数 **k，**你需要找到该数组中和为 **k** 的连续的子数组的个数。
>
>   **示例 1 :**
>
>   ```
>   输入:nums = [1,1,1], k = 2
>   输出: 2 , [1,1] 与 [1,1] 为两种不同的情况。
>   ```
>
>   **说明 :**
>
>   1. 数组的长度为 [1, 20,000]。
>   2. 数组中元素的范围是 [-1000, 1000] ，且整数 **k** 的范围是 [-1e7, 1e7]。

```java
 public int subarraySum(int[] nums, int k) {
        int count = 0;
        for (int start = 0; start < nums.length; ++start) {
            int sum = 0;
            for (int end = start; end < nums.length; end++) {
                sum += nums[end];
                if (sum == k) {
                    count++;
                }
            }
        }
        return count;
    }
```

# [406. 根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height/)

>   假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对 (h, k) 表示，其中 h 是这个人的身高，k 是应该排在这个人前面且身高大于或等于 h 的人数。 例如：[5,2] 表示前面应该有 2 个身高大于等于 5 的人，而 [5,0] 表示前面不应该存在身高大于等于 5 的人。
>
>   编写一个算法，根据每个人的身高 h 重建这个队列，使之满足每个整数对 (h, k) 中对人数 k 的要求。
>
>   示例：
>
>   输入：[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]
>   输出：[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]

```java
 public int[][] reconstructQueue(int[][] people) {
         if (0 == people.length || 0 == people[0].length)
            return new int[0][0];
         //按照身高降序 K升序排序 
        Arrays.sort(people, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[0] == o2[0] ? o1[1] - o2[1] : o2[0] - o1[0];
            }
        });
        List<int[]> list = new ArrayList<>();
        //K值定义为 排在h前面且身高大于或等于h的人数 
        //因为从身高降序开始插入，此时所有人身高都大于等于h
        //因此K值即为需要插入的位置
        for (int[] i : people) {
            list.add(i[1], i);
        }
        return list.toArray(new int[list.size()][]);
    }
```

# [47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)

>   给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列。
>
>    
>
>   示例 1：
>
>   输入：nums = [1,1,2]
>   输出：
>   [[1,1,2],
>    [1,2,1],
>    [2,1,1]]
>   示例 2：
>
>   输入：nums = [1,2,3]
>   输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

```java
public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> res = new LinkedList<>();
        List<Integer> list = new ArrayList<>();
        int len = nums.length;
        for (int i : nums) {
            list.add(i);
        }
        getPermute(res, list, 0, len);
        return res;
    }
        public static void getPermute(List<List<Integer>> lists, List<Integer> list, int begin, int len) {
        if (begin == len) {
            if (!lists.contains(new ArrayList<>(list))) {
                lists.add(new ArrayList<>(list));
            }
        } else {
            for (int i = begin; i < len; i++) {
                Collections.swap(list, begin, i);
                getPermute(lists, list, begin + 1, len);
                Collections.swap(list, begin, i);
            }
        }
    }
```

# [120. 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

>   
>   给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。
>
>   **相邻的结点** 在这里指的是 `下标` 与 `上一层结点下标` 相同或者等于 `上一层结点下标 + 1` 的两个结点。
>
>    
>
>   例如，给定三角形：
>
>   ```
>   [
>        [2],
>       [3,4],
>      [6,5,7],
>     [4,1,8,3]
>   ]
>   ```
>
>   自顶向下的最小路径和为 `11`（即，**2** + **3** + **5** + **1** = 11）。

```java
   public int minimumTotal(List<List<Integer>> triangle) {
        //如何优化成只用一维数组呢
        int n = triangle.size();
        int[][] dp = new int[n][n];
        dp[0][0] = triangle.get(0).get(0);
        for(int i = 1;i < n;i++){
            //注意是get0
            dp[i][0] = dp[i-1][0] + triangle.get(i).get(0);
            //注意是j
            for(int j = 1;j < i;j++){
                dp[i][j] = Math.min(dp[i-1][j-1],dp[i-1][j]) + triangle.get(i).get(j);
            }
            //
            dp[i][i] = dp[i-1][i-1] + triangle.get(i).get(i);
        }
        int min = dp[n-1][0];
        for(int i = 0;i< n;i++){
            min = Math.min(dp[n-1][i],min);
        }
        return min;
    }
```

```java
 int n = triangle.size();
    int[] f = new int[n];
    f[0] = triangle.get(0).get(0);
    for (int i = 1; i < n; ++i) {
        f[i] = f[i - 1] + triangle.get(i).get(i);
        for (int j = i - 1; j > 0; --j) {
            f[j] = Math.min(f[j - 1], f[j]) + triangle.get(i).get(j);
        }
        f[0] += triangle.get(i).get(0);
    }
    int minTotal = f[0];
    for (int i = 1; i < n; ++i) {
        minTotal = Math.min(minTotal, f[i]);
    }
    return minTotal;
}
```

# [179. 最大数](https://leetcode-cn.com/problems/largest-number/)

>   给定一组非负整数 nums，重新排列它们每个数字的顺序（每个数字不可拆分）使之组成一个最大的整数。
>
>   注意：输出结果可能非常大，所以你需要返回一个字符串而不是整数。
>

```java
 public String largestNumber(int[] nums) {
      String[] h = new String[nums.length];
        for (int i = 0; i < nums.length; i++) h[i] = String.valueOf(nums[i]);
        Arrays.sort(h, (a, b) -> {
            String l1 = a + b;
            String l2 = b + a;
            return l2.compareTo(l1);
        });
        if (h[0].charAt(0) == '0') return "0";
        StringBuilder sb = new StringBuilder();
        for (String ky : h) sb.append(ky);
        return sb.toString();
    }
```

# [185. 部门工资前三高的所有员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

>   `Employee` 表包含所有员工信息，每个员工有其对应的工号 `Id`，姓名 `Name`，工资 `Salary` 和部门编号 `DepartmentId` 。

```sql
select d.Name as Department,e.Name as Employee,e.Salary as Salary
from Employee as e left join Department as d 
on e.DepartmentId = d.Id
where e.Id in
(
    select e1.Id
    from Employee as e1 left join Employee as e2
    on e1.DepartmentId = e2.DepartmentId and e1.Salary < e2.Salary
    group by e1.Id
    having count(distinct e2.Salary) <= 2
)
and e.DepartmentId in (select Id from Department)
order by d.Id asc,e.Salary desc
```

#[386. 字典序排数](https://leetcode-cn.com/problems/lexicographical-numbers/) 

>   给定一个整数 n, 返回从 1 到 n 的字典顺序。
>
>   例如，
>
>   给定 n =1 3，返回 [1,10,11,12,13,2,3,4,5,6,7,8,9] 。
>
>   请尽可能的优化算法的时间复杂度和空间复杂度。 输入的数据 n 小于等于 5,000,000。
>

```java
  public List<Integer> lexicalOrder(int n) {
        ArrayList<Integer> res = new ArrayList<>();
        for (int i = 1; i < 10; i++) 
            dfs(n, i, res);
        return res;
    }
    private static void dfs(int n, int target, List<Integer> list) {
        if (target>n) return;
        list.add(target);
        target*=10;
        for (int i = 0; i < 10; i++) 
            dfs(n,target+i,list);
    }
```

# [460. LFU 缓存](https://leetcode-cn.com/problems/lfu-cache/)

>   请你为 最不经常使用（LFU）缓存算法设计并实现数据结构。
>
>   实现 LFUCache 类：
>
>   LFUCache(int capacity) - 用数据结构的容量 capacity 初始化对象
>   int get(int key) - 如果键存在于缓存中，则获取键的值，否则返回 -1。
>   void put(int key, int value) - 如果键已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量时，则应该在插入新项之前，使最不经常使用的项无效。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 最久未使用 的键。
>   注意「项的使用次数」就是自插入该项以来对其调用 get 和 put 函数的次数之和。使用次数会在对应项被移除后置为 0 。

```java
class LFUCache {
    // 缓存容量，时间戳
    int capacity, time;
    Map<Integer, Node> key_table;
    TreeSet<Node> S;

    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.time = 0;
        key_table = new HashMap<Integer, Node>();
        S = new TreeSet<Node>();
    }
    
    public int get(int key) {
        if (capacity == 0) {
            return -1;
        }
        // 如果哈希表中没有键 key，返回 -1
        if (!key_table.containsKey(key)) {
            return -1;
        }
        // 从哈希表中得到旧的缓存
        Node cache = key_table.get(key);
        // 从平衡二叉树中删除旧的缓存
        S.remove(cache);
        // 将旧缓存更新
        cache.cnt += 1;
        cache.time = ++time;
        // 将新缓存重新放入哈希表和平衡二叉树中
        S.add(cache);
        key_table.put(key, cache);
        return cache.value;
    }
    
    public void put(int key, int value) {
        if (capacity == 0) {
            return;
        }
        if (!key_table.containsKey(key)) {
            // 如果到达缓存容量上限
            if (key_table.size() == capacity) {
                // 从哈希表和平衡二叉树中删除最近最少使用的缓存
                key_table.remove(S.first().key);
                S.remove(S.first());
            }
            // 创建新的缓存
            Node cache = new Node(1, ++time, key, value);
            // 将新缓存放入哈希表和平衡二叉树中
            key_table.put(key, cache);
            S.add(cache);
        } else {
            // 这里和 get() 函数类似
            Node cache = key_table.get(key);
            S.remove(cache);
            cache.cnt += 1;
            cache.time = ++time;
            cache.value = value;
            S.add(cache);
            key_table.put(key, cache);
        }
    }
}

class Node implements Comparable<Node> {
    int cnt, time, key, value;

    Node(int cnt, int time, int key, int value) {
        this.cnt = cnt;
        this.time = time;
        this.key = key;
        this.value = value;
    }

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof Node) {
            Node rhs = (Node) anObject;
            return this.cnt == rhs.cnt && this.time == rhs.time;
        }
        return false;
    }

    public int compareTo(Node rhs) {
        return cnt == rhs.cnt ? time - rhs.time : cnt - rhs.cnt;
    }

    public int hashCode() {
        return cnt * 1000000007 + time;
    }
```

# [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

>   给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。
>
>   为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。
>
>   说明：不允许修改给定的链表。
>
>   进阶：
>
>   你是否可以使用 O(1) 空间解决此题？
>

```java
  public ListNode detectCycle(ListNode head) {
        if(head == null) return null;
        ListNode slow = head;
        ListNode fast = head;
        boolean cycle = false;
        while(fast.next != null && fast.next.next != null){
            fast = fast.next.next;
            slow = slow.next;
            
            if(fast == slow){
                cycle = true;
                break;
            }
        }
        if(cycle){
            ListNode first = head;
            while(first != slow){
                slow  = slow.next;
                first = first.next;
            }
            return first;
        }
        return null;
    }
```

# [547. 朋友圈](https://leetcode-cn.com/problems/friend-circles/)

>   班上有 N 名学生。其中有些人是朋友，有些则不是。他们的友谊具有是传递性。如果已知 A 是 B 的朋友，B 是 C 的朋友，那么我们可以认为 A 也是 C 的朋友。所谓的朋友圈，是指所有朋友的集合。
>
>   给定一个 N * N 的矩阵 M，表示班级中学生之间的朋友关系。如果M[i][j] = 1，表示已知第 i 个和 j 个学生互为朋友关系，否则为不知道。你必须输出所有学生中的已知的朋友圈总数。
>
>   示例 1：
>
>   输入：
>   [[1,1,0],
>    [1,1,0],
>    [0,0,1]]
>   输出：2 
>   解释：已知学生 0 和学生 1 互为朋友，他们在一个朋友圈。
>   第2个学生自己在一个朋友圈。所以返回 2 。
>   示例 2：
>
>   输入：
>   [[1,1,0],
>    [1,1,1],
>    [0,1,1]]
>   输出：1
>   解释：已知学生 0 和学生 1 互为朋友，学生 1 和学生 2 互为朋友，所以学生 0 和学生 2 也是朋友，所以他们三个在一个朋友圈，返回 1 。

```java
  public void dfs(int[][] M, int[] visited, int i) {
        for (int j = 0; j < M.length; j++) {
            if (M[i][j] == 1 && visited[j] == 0) {
                visited[j] = 1;
                dfs(M, visited, j);
            }
        }
    }
    public int findCircleNum(int[][] M) {
        int[] visited = new int[M.length];
        int count = 0;
        for (int i = 0; i < M.length; i++) {
            if (visited[i] == 0) {
                dfs(M, visited, i);
                count++;
            }
        }
        return count;
    }
```

# [61. 旋转链表](https://leetcode-cn.com/problems/rotate-list/)

>   给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。
>
>   示例 1:
>
>   输入: 1->2->3->4->5->NULL, k = 2
>   输出: 4->5->1->2->3->NULL
>   解释:
>   向右旋转 1 步: 5->1->2->3->4->NULL
>   向右旋转 2 步: 4->5->1->2->3->NULL
>   示例 2:
>
>   输入: 0->1->2->NULL, k = 4
>   输出: 2->0->1->NULL
>   解释:
>   向右旋转 1 步: 2->0->1->NULL
>   向右旋转 2 步: 1->2->0->NULL
>   向右旋转 3 步: 0->1->2->NULL
>   向右旋转 4 步: 2->0->1->NULL

```java
public ListNode rotateRight(ListNode head, int k) {
            if(head==null||k==0){
        return head;
    }
    ListNode cursor=head;
    ListNode tail=null;//尾指针
    int length=1;
    while(cursor.next!=null)//循环 得到总长度
    {
        cursor=cursor.next;
        length++;
    }
    int loop=length-(k%length);//得到循环的次数
    tail=cursor;//指向尾结点
    cursor.next=head;//改成循环链表
    cursor=head;//指向头结点
    for(int i=0;i<loop;i++){//开始循环
        cursor=cursor.next;
        tail=tail.next;
    }
    tail.next=null;//改成单链表
    return cursor;//返回当前头
    }
```

# [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

>   给定一个二叉树，返回它的 后序 遍历。
>
>   示例:
>
>   输入: [1,null,2,3]  
>      1
>       \
>        2
>       /
>      3 
>
>   输出: [3,2,1]
>

```java
   public List<Integer> postorderTraversal(TreeNode root) {
         LinkedList<TreeNode> stack = new LinkedList<>();
        LinkedList<Integer> output = new LinkedList<>();
        if (root == null) {
            return output;
        }
        stack.add(root);
        while (!stack.isEmpty()) {
            TreeNode node = stack.pollLast();
            output.addFirst(node.val);
            if (node.left != null) {
                stack.add(node.left);
            }
            if (node.right != null) {
                stack.add(node.right);
            }
        }
        return output;
    }
```

# [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

>   
>   假设按照升序排序的数组在预先未知的某个点上进行了旋转。例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` 。
>
>   请找出其中最小的元素。
>
>    
>
>   **示例 1：**
>
>   ```
>   输入：nums = [3,4,5,1,2]
>   输出：1
>   ```
>
>   **示例 2：**
>
>   ```
>   输入：nums = [4,5,6,7,0,1,2]
>   输出：0
>   ```
>
>   **示例 3：**
>
>   ```
>   输入：nums = [1]
>   输出：1
>   ```

```java
 public int findMin(int[] nums) {
        int l = 0;
        int r = nums.length - 1;
        while(l < r){
            int mid = l + (r - l) / 2;
            if(nums[mid] < nums[r]){
                r = mid;
            }else if(nums[mid] > nums[r]){
                l = mid + 1;
            }else{
                return nums[mid];
            }
        }
        return nums[l];
    }
```

# [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

>   给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度。
>
>   一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>   例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。
>
>   若这两个字符串没有公共子序列，则返回 0。
>
>    
>
>   示例 1:
>
>   输入：text1 = "abcde", text2 = "ace" 
>   输出：3  
>   解释：最长公共子序列是 "ace"，它的长度为 3。
>   示例 2:
>
>   输入：text1 = "abc", text2 = "abc"
>   输出：3
>   解释：最长公共子序列是 "abc"，它的长度为 3。
>   示例 3:
>
>   输入：text1 = "abc", text2 = "def"
>   输出：0
>   解释：两个字符串没有公共子序列，返回 0。

```java
    public int longestCommonSubsequence(String text1, String text2) {
               int len1 = text1.length();
        int len2 = text2.length();
        if (len1 == 0 || len2 == 0) {
            return 0;
        }
        char[] t1 = text1.toCharArray();
        char[] t2 = text2.toCharArray();
        int[] score = new int[len1+1];
        int last = 0;
        // System.out.println(score);
        // 初始化
        Arrays.fill(score, 0);
        for (int i = 0; i < len2; i++) {
            last = 0;
            for (int j = 1; j < len1+1; j++) {
                // 状态转移函数
                int temp = score[j];
                if (t1[j-1] == t2[i]) {
                    score[j] = last + 1;
                    // System.out.println(last);
                    // System.out.println(t1[j-1]);
                }else if (score[j] < score[j-1]) {
                    score[j] = score[j-1];
                }
                last = temp;
            }
            //System.out.println(score);
        }
        return score[len1];
    }
```

# [35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

>   示例 1:
>
>   输入: [1,3,5,6], 5
>   输出: 2
>   示例 2:
>
>   输入: [1,3,5,6], 2
>   输出: 1
>   示例 3:
>
>   输入: [1,3,5,6], 7
>   输出: 4
>   示例 4:
>
>   输入: [1,3,5,6], 0
>   输出: 0

```java
 public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length;
        //二分法
        while(left < right){
            int mid = left + (right - left)/2;
            if(nums[mid] == target){
                return mid;
                //中间小于目标值,舍去左边
            }else if(nums[mid] < target){
                left = mid + 1;
            }else{
                //中间大于目标值,舍去右边
                right = mid;
            }
        }
        return left;
    }
```

#[108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

>   将一个按照升序排列的有序数组，转换为一棵高度平衡二叉搜索树。
>
>   本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。
>
>   示例:
>
>   给定有序数组: [-10,-3,0,5,9],
>
>   一个可能的答案是：[0,-3,9,-10,null,5]，它可以表示下面这个高度平衡二叉搜索树：
>
>         0
>        / \
>      -3   9
>      /   /
>    -10  5

```java
   public TreeNode sortedArrayToBST(int[] nums) {
        if(nums == null || nums.length == 0){
            return null;
        }
        return help(nums,0,nums.length - 1);
    }
    private TreeNode help(int[] nums,int low,int high){
        if(low > high) return null;
        int mid = (high + low)/2;
        TreeNode tree = new TreeNode(nums[mid]);
        tree.left = help(nums,low,mid-1);
        tree.right = help(nums,mid + 1,high);
        return tree;
    }
```

# [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

>   给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的 连续 子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。
>
>    
>
>   示例：
>
>   输入：s = 7, nums = [2,3,1,2,4,3]
>   输出：2
>   解释：子数组 [4,3] 是该条件下的长度最小的子数组。
>

```java

```

# [剑指 Offer 07 - 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

>  输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
>
>   
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
>
>
>  限制：
>
>  0 <= 节点个数 <= 5000
>

```java
     int n = preorder.length;
        if (n == 0)
            return null;
        int rootVal = preorder[0], rootIndex = 0;
        for (int i = 0; i < n; i++) {
            if (inorder[i] == rootVal) {
                rootIndex = i;
                break;
            }
        }
        TreeNode root = new TreeNode(rootVal);
        root.left = buildTree(Arrays.copyOfRange(preorder, 1, 1 + rootIndex),
                              Arrays.copyOfRange(inorder, 0, rootIndex));
        root.right = buildTree(Arrays.copyOfRange(preorder, 1 + rootIndex, n),
                               Arrays.copyOfRange(inorder, rootIndex + 1, n));

        return root;
    }
```

# [35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

>  给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
>  你可以假设数组中无重复元素。
>
>  示例 1:
>
>  输入: [1,3,5,6], 5
>  输出: 2
>  示例 2:
>
>  输入: [1,3,5,6], 2
>  输出: 1
>  示例 3:
>
>  输入: [1,3,5,6], 7
>  输出: 4
>  示例 4:
>
>  输入: [1,3,5,6], 0
>  输出: 0

```java
    public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length;
        //二分法
        while(left < right){
            int mid = left + (right - left)/2;
            if(nums[mid] == target){
                return mid;
                //中间小于目标值,舍去左边
            }else if(nums[mid] < target){
                left = mid + 1;
            }else{
                //中间大于目标值,舍去右边
                right = mid;
            }
        }
        return left;
    }
```

# [48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)

>  给定一个 n × n 的二维矩阵表示一个图像。
>
> 将图像顺时针旋转 90 度。
>
> 说明：
>
> 你必须在原地旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要使用另一个矩阵来旋转图像。
>
> 示例 1:
>
> 给定 matrix = 
> [
>   [1,2,3],
>   [4,5,6],
>   [7,8,9]
> ],
>
> 原地旋转输入矩阵，使其变为:
> [
>   [7,4,1],
>   [8,5,2],
>   [9,6,3]
> ]

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;

    // transpose matrix
    for (int i = 0; i < n; i++) {
      for (int j = i; j < n; j++) {
        int tmp = matrix[j][i];
        matrix[j][i] = matrix[i][j];
        matrix[i][j] = tmp;
      }
    }
    // reverse each row
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < n / 2; j++) {
        int tmp = matrix[i][j];
        matrix[i][j] = matrix[i][n - j - 1];
        matrix[i][n - j - 1] = tmp;
      }
}
```

