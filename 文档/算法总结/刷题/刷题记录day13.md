[toc]

# [384. 打乱数组](https://leetcode-cn.com/problems/shuffle-an-array/)

>   给你一个整数数组 nums ，设计算法来打乱一个没有重复元素的数组。
>
>   实现 Solution class:
>
>   Solution(int[] nums) 使用整数数组 nums 初始化对象
>   int[] reset() 重设数组到它的初始状态并返回
>   int[] shuffle() 返回数组随机打乱后的结果
>
>
>   示例：
>
>   输入
>   ["Solution", "shuffle", "reset", "shuffle"]
>   [[[1, 2, 3]], [], [], []]
>   输出
>   [null, [3, 1, 2], [1, 2, 3], [1, 3, 2]]
>
>   解释
>   Solution solution = new Solution([1, 2, 3]);
>   solution.shuffle();    // 打乱数组 [1,2,3] 并返回结果。任何 [1,2,3]的排列返回的概率应该相同。例如，返回 [3, 1, 2]
>   solution.reset();      // 重设数组到它的初始状态 [1, 2, 3] 。返回 [1, 2, 3]
>   solution.shuffle();    // 随机返回数组 [1, 2, 3] 打乱后的结果。例如，返回 [1, 3, 2]

**Fisher-Yates 洗牌算法**

```java
class Solution {
    private int[] nums;
  	//保存原始数组
    private int[] originalNums;

    public Solution(int[] nums) {
        this.nums = nums;
        this.originalNums = Arrays.copyOf(nums, nums.length);
    }

    /**
     * Resets the array to its original configuration and return it.
     */
    public int[] reset() {
        return this.originalNums;
    }

    /**
     * Returns a random shuffling of the array.
     */
    public int[] shuffle() {
        Random random = new Random();
      	//
        for (int i = 0; i < nums.length / 2; i++) {
            // 每次只需拿第一个元素进行交换即可
          	// 因为第一个元素每次都在变化
            swap(nums, 0, random.nextInt(nums.length));
        }
        return nums;
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

#[1044. 最长重复子串](https://leetcode-cn.com/problems/longest-duplicate-substring/) 

>   给出一个字符串 S，考虑其所有重复子串（S 的连续子串，出现两次或多次，可能会有重叠）。
>
>   返回任何具有最长可能长度的重复子串。（如果 S 不含重复子串，那么答案为 ""。）
>
>    
>
>   示例 1：
>
>   输入："banana"
>   输出："ana"
>   示例 2：
>
>   输入："abcd"
>   输出：""

**Rabin-Karp算法**

```java
 /*
    Rabin-Karp with polynomial rolling hash.
        Search a substring of given length
        that occurs at least 2 times.
        Return start position if the substring exits and -1 otherwise.
        */
    public int search(int L, int a, long modulus, int n, int[] nums) {
        // compute the hash of string S[:L]
        long h = 0;
        for (int i = 0; i < L; ++i) h = (h * a + nums[i]) % modulus;

        // already seen hashes of strings of length L
        HashSet<Long> seen = new HashSet();
        seen.add(h);
        // const value to be used often : a**L % modulus
        long aL = 1;
        for (int i = 1; i <= L; ++i) aL = (aL * a) % modulus;

        for (int start = 1; start < n - L + 1; ++start) {
            // compute rolling hash in O(1) time
            h = (h * a - nums[start - 1] * aL % modulus + modulus) % modulus;
            h = (h + nums[start + L - 1]) % modulus;
            if (seen.contains(h)) return start;
            seen.add(h);
        }
        return -1;
    }

    public String longestDupSubstring(String S) {
        int n = S.length();
        // convert string to array of integers
        // to implement constant time slice
        int[] nums = new int[n];
        for (int i = 0; i < n; ++i) nums[i] = (int)S.charAt(i) - (int)'a';
        // base value for the rolling hash function
        int a = 26;
        // modulus value for the rolling hash function to avoid overflow
        long modulus = (long)Math.pow(2, 32);

        // binary search, L = repeating string length
        int left = 1, right = n;
        int L;
        while (left != right) {
            L = left + (right - left) / 2;
            if (search(L, a, modulus, n, nums) != -1) left = L + 1;
            else right = L;
        }

        int start = search(left - 1, a, modulus, n, nums);
        return start != -1 ? S.substring(start, start + left - 1) : "";
    }
}
```

#[416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/) 

>   给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。
>
>   注意:
>
>   每个数组中的元素不会超过 100
>   数组的大小不会超过 200
>   示例 1:
>
>   输入: [1, 5, 11, 5]
>
>   输出: true
>
>   解释: 数组可以分割成 [1, 5, 5] 和 [11].
>
>
>   示例 2:
>
>   输入: [1, 2, 3, 5]
>
>   输出: false
>
>   解释: 数组不能分割成两个元素和相等的子集.

[NP 完全问题](https://baike.baidu.com/item/NP完全问题)

```java
 public boolean canPartition(int[] nums) {
        int n = nums.length;
        if (n < 2) {
            return false;
        }
        int sum = 0, maxNum = 0;
        for (int num : nums) {
            sum += num;
            maxNum = Math.max(maxNum, num);
        }
        if (sum % 2 != 0) {
            return false;
        }
        int target = sum / 2;
        if (maxNum > target) {
            return false;
        }
        boolean[][] dp = new boolean[n][target + 1];
        for (int i = 0; i < n; i++) {
            dp[i][0] = true;
        }
        dp[0][nums[0]] = true;
        for (int i = 1; i < n; i++) {
            int num = nums[i];
            for (int j = 1; j <= target; j++) {
                if (j >= num) {
                    dp[i][j] = dp[i - 1][j] | dp[i - 1][j - num];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n - 1][target];
    }
```

#[剑指 Offer 10- I - 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/) 

>   写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：
>
>   F(0) = 0,   F(1) = 1
>   F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
>   斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。
>
>   答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。
>

```java
 public int fib(int n) {
        if (n <= 1) return n;
        int first = 0;
        int second = 1;
        int result = 0;
        while (--n > 0) {
            result = first + second;
            if (result >= 1000000007) {
                result -= 1000000007;
            }
            first = second;
            second = result;
        }
        return result;
    }
```

#[面试题 10.01 - 合并排序的数组](https://leetcode-cn.com/problems/sorted-merge-lcci/) 

>   
>   给定两个排序后的数组 A 和 B，其中 A 的末端有足够的缓冲空间容纳 B。 编写一个方法，将 B 合并入 A 并排序。
>
>   初始化 A 和 B 的元素数量分别为 *m* 和 *n*。
>
>   **示例:**
>
>   ```java
>   输入:
>   A = [1,2,3,0,0,0], m = 3
>   B = [2,5,6],       n = 3
>   
>   输出: [1,2,2,3,5,6]
>   ```
>
>   - `A.length == n + m`

```java
 public void merge(int[] A, int m, int[] B, int n) {
        int k = m + n - 1, i = m - 1, j = n - 1;
        while (i >= 0 && j >= 0) {
            if (A[i] < B[j]) {
                A[k--] = B[j--];
            } else {
                A[k--] = A[i--];
            }
        }
        while (j >= 0) A[k--] = B[j--];
    }
```

#[面试题 03.02 - 栈的最小值](https://leetcode-cn.com/problems/min-stack-lcci/) 

>   
>   请设计一个栈，除了常规栈支持的pop与push函数以外，还支持min函数，该函数返回栈元素中的最小值。执行push、pop和min操作的时间复杂度必须为O(1)。
>
>   
>
>   **示例：**
>
>   ```java
>   MinStack minStack = new MinStack();
>   minStack.push(-2);
>   minStack.push(0);
>   minStack.push(-3);
>   minStack.getMin();   --> 返回 -3.
>   minStack.pop();
>   minStack.top();      --> 返回 0.
>   minStack.getMin();   --> 返回 -2.
>   ```

```java
int[] imitate;
    int[] min;
    int top = -1;

    /** initialize your data structure here. **/
    public MinStack() {
        imitate = new int[10000];
        min = new int[10000];      

    }
    
    public void push(int x) {
        if(top == -1) 
        min[top + 1] = x;

        else if(min[top] > x)
        {
            min[top + 1] = x;
        }
        else
        {
            min[top + 1] = min[top];
        }
        top++;
        imitate[top] = x;

    }
    
    public void pop() {
        top--;

    }
    
    public int top() {
        return imitate[top];

    }
    
    public int getMin() {
        return min[top];

    }
```

#[剑指 Offer 46 - 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/) 

>   给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。
>
>    
>
>   示例 1:
>
>   输入: 12258
>   输出: 5
>   解释: 12258有5种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"
>
>
>   提示：
>
>   0 <= num < 231
>

```java
 public int translateNum(int num) {
        if (num <= 9) return 1;
        int ba = num % 100;
        return (ba <= 9 || ba >= 26)
                ? translateNum(num / 10)
                : translateNum(num / 10) + translateNum(num / 100);
    }
```

#[343. 整数拆分](https://leetcode-cn.com/problems/integer-break/) 

>   给定一个正整数 n，将其拆分为至少两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。
>
>   示例 1:
>
>   输入: 2
>   输出: 1
>   解释: 2 = 1 + 1, 1 × 1 = 1。
>   示例 2:
>
>   输入: 10
>   输出: 36
>   解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
>   说明: 你可以假设 n 不小于 2 且不大于 58。

```java
  public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        dp[1] = 1;
        for(int i = 2; i <= n; i++){
            for(int j = 1; j <= i - 1; j++){
                dp[i] = Math.max(dp[i], Math.max(j * dp[i - j], j * (i - j)));
            }
        }
        return dp[n];
    }
```

#[410. 分割数组的最大值](https://leetcode-cn.com/problems/split-array-largest-sum/) 

>   给定一个非负整数数组和一个整数 m，你需要将这个数组分成 m 个非空的连续子数组。设计一个算法使得这 m 个子数组各自和的最大值最小。
>
>   注意:
>   数组长度 n 满足以下条件:
>
>   1 ≤ n ≤ 1000
>   1 ≤ m ≤ min(50, n)
>   示例:
>
>   输入:
>   nums = [7,2,5,10,8]
>   m = 2
>
>   输出:
>   18
>
>   解释:
>   一共有四种方法将nums分割为2个子数组。
>   其中最好的方式是将其分为[7,2,5] 和 [10,8]，
>   因为此时这两个子数组各自的和的最大值为18，在所有情况中最小。

```java
public int splitArray(int[] nums, int m) {
        int left = 0;
        int right = 0;
        for(int num : nums){
            left = Math.max(left, num);
            right += num;
        }
         //二分 logn，内部遍历数组 n，时间复杂度 O(nlogn)
        while(left < right){
            int cnt = 1;
            int mid = (left + right) >>> 1;
            int sum = 0;
            for(int num : nums){
                if(sum + num > mid){
                    sum = 0;
                    cnt++;
                }
                sum += num;
            }
            if(cnt > m){
                left = mid + 1;
            }else{
                right = mid;
            }
        }
        return left;
    }
```

#[253. 会议室 II](https://leetcode-cn.com/problems/meeting-rooms-ii/) 

>   给定一个会议时间安排的数组，每个会议时间都会包括开始和结束的时间 [[s1,e1],[s2,e2],...] (si < ei)，为避免会议冲突，同时要考虑充分利用会议室资源，请你计算至少需要多少间会议室，才能满足这些会议安排。
>
>   示例 1:
>
>   输入: [[0, 30],[5, 10],[15, 20]]
>   输出: 2
>   示例 2:
>
>   输入: [[7,10],[2,4]]
>   输出: 1

```java
public int minMeetingRooms(int[][] intervals) {
        if(intervals == null || intervals.length == 0) return 0; 
        int[] start = new int[intervals.length];
        int[] end = new int[intervals.length];
        for(int i=0;i<intervals.length;i++){
            start[i] = intervals[i][0];
            end[i] = intervals[i][1];
        }
        Arrays.sort(start);
        Arrays.sort(end);
        
        int rooms=0, activeMeeting = 0;
        int i=0,j=0;
        while(i<intervals.length && j<intervals.length){
            if(start[i]<end[j]){
                activeMeeting++;
                i++;
            }else{
                activeMeeting--;
                j++;
         }
            rooms = Math.max(rooms,activeMeeting);
        }
        return rooms;
    }
```

#[224. 基本计算器](https://leetcode-cn.com/problems/basic-calculator/) 

>   实现一个基本的计算器来计算一个简单的字符串表达式的值。
>
>   字符串表达式可以包含左括号 ( ，右括号 )，加号 + ，减号 -，非负整数和空格  。
>
>   示例 1:
>
>   输入: "1 + 1"
>   输出: 2
>   示例 2:
>
>   输入: " 2-1 + 2 "
>   输出: 3
>   示例 3:
>
>   输入: "(1+(4+5+2)-3)+(6+8)"
>   输出: 23

```java
//基本计算器
    public static int calculate(String s) {
        return process(s.toCharArray(), 0)[0];
    }

    private static int[] process(char[] chs, int index) {
        int res = 0;
        char opera = '+';
        int first = 0; // opera的前一个数
        int second = 0; // opera的后一个数
        while (index < chs.length && chs[index] != ')') {
            if (chs[index] >= '0' && chs[index] <= '9') {
                second = second * 10 + (int) chs[index++] - '0';
            } else if (chs[index] != '(' && chs[index] != ' ') {
                if (opera == '+' || opera == '-') {
                    res += first;
                    first = opera == '+' ? second : -second;
                } else {
                    first = opera == '*' ? first * second : first / second;
                }
                opera = chs[index++];
                second = 0;
            } else if (chs[index] == '(') {
                int[] info = process(chs, index + 1);
                second = info[0];
                index = info[1] + 1;
            } else {
                ++index;
            }
        }
        if (opera == '+' || opera == '-') {
            res += opera == '+' ? first + second : first - second;
        } else {
            res += opera == '*' ? first * second : first / second;
        }
        return new int[]{res, index};
    }
```

#[518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/) 

>   给定不同面额的硬币和一个总金额。写出函数来计算可以凑成总金额的硬币组合数。假设每一种面额的硬币有无限个。 
>
>    
>
>   示例 1:
>
>   输入: amount = 5, coins = [1, 2, 5]
>   输出: 4
>   解释: 有四种方式可以凑成总金额:
>   5=5
>   5=2+2+1
>   5=2+1+1+1
>   5=1+1+1+1+1
>   示例 2:
>
>   输入: amount = 3, coins = [2]
>   输出: 0
>   解释: 只用面额2的硬币不能凑成总金额3。
>   示例 3:
>
>   输入: amount = 10, coins = [10] 
>   输出: 1

```java
  public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
         dp[0] = 1;

    for (int coin : coins) {
      for (int x = coin; x < amount + 1; ++x) {
        dp[x] += dp[x - coin];
      }
    }
    return dp[amount];
  }
```

#[316. 去除重复字母](https://leetcode-cn.com/problems/remove-duplicate-letters/) 

>   给你一个字符串 s ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 返回结果的字典序最小（要求不能打乱其他字符的相对位置）。
>
>   注意：该题与 1081 https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters 相同
>
>    
>
>   示例 1：
>
>   输入：s = "bcabc"
>   输出："abc"
>   示例 2：
>
>   输入：s = "cbacdcbc"
>   输出："acdb"

```java
 public String removeDuplicateLetters(String s) {
        Stack<Character> stack=new Stack<>();
        for (int i = 0; i < s.length(); i++) {
            Character c=s.charAt(i);
            if(stack.contains(c))
                continue;
            while(!stack.isEmpty()&&stack.peek()>c&&s.indexOf(stack.peek(),i)!=-1)
                stack.pop();
            stack.push(c);
        }
        char chars[]=new char[stack.size()];
        for (int i = 0; i < stack.size(); i++) {
            chars[i]=stack.get(i);
        }
        return new String(chars);
    }
```

#[86. 分隔链表](https://leetcode-cn.com/problems/partition-list/) 

>   给定一个链表和一个特定值 x，对链表进行分隔，使得所有小于 x 的节点都在大于或等于 x 的节点之前。
>
>   你应当保留两个分区中每个节点的初始相对位置。
>
>    
>
>   示例:
>
>   输入: head = 1->4->3->2->5->2, x = 3
>   输出: 1->2->2->4->3->5

```java
 public ListNode partition(ListNode head, int x) {
        ListNode dummyHead1 = new ListNode(0);
        ListNode dummyHead2 = new ListNode(0);
        ListNode node1 = dummyHead1;
        ListNode node2 = dummyHead2;
        while (head != null) {
            if (head.val < x) {
                node1.next = head;
                head = head.next;
                node1 = node1.next;
                node1.next = null;
            } else {
                node2.next = head;
                head = head.next;
                node2 = node2.next;
                node2.next = null;
            }
        }
        node1.next = dummyHead2.next;
        return dummyHead1.next;
    }
```

#[351. 安卓系统手势解锁](https://leetcode-cn.com/problems/android-unlock-patterns/) 

>   我们都知道安卓有个手势解锁的界面，是一个 3 x 3 的点所绘制出来的网格。用户可以设置一个 “解锁模式” ，通过连接特定序列中的点，形成一系列彼此连接的线段，每个线段的端点都是序列中两个连续的点。如果满足以下两个条件，则 k 点序列是有效的解锁模式：
>
>   解锁模式中的所有点 互不相同 。
>   假如模式中两个连续点的线段需要经过其他点，那么要经过的点必须事先出现在序列中（已经经过），不能跨过任何还未被经过的点。
>
>
>   以下是一些有效和无效解锁模式的示例：
>
>   
>
>
>   无效手势：[4,1,3,6] ，连接点 1 和点 3 时经过了未被连接过的 2 号点。
>   无效手势：[4,1,9,2] ，连接点 1 和点 9 时经过了未被连接过的 5 号点。
>   有效手势：[2,4,1,3,6] ，连接点 1 和点 3 是有效的，因为虽然它经过了点 2 ，但是点 2 在该手势中之前已经被连过了。
>   有效手势：[6,5,4,1,9,2] ，连接点 1 和点 9 是有效的，因为虽然它经过了按键 5 ，但是点 5 在该手势中之前已经被连过了。
>   给你两个整数，分别为 m 和 n ，那么请你统计一下有多少种 不同且有效的解锁模式 ，是 至少 需要经过 m 个点，但是 不超过 n 个点的。
>
>   两个解锁模式 不同 需满足：经过的点不同或者经过点的顺序不同。
>
>    
>
>   示例 1：
>
>   输入：m = 1, n = 1
>   输出：9
>   示例 2：
>
>   输入：m = 1, n = 2
>   输出：65

```java
		private boolean used[] = new boolean[9];

    public int numberOfPatterns(int m, int n) {	        
        int res = 0;
        for (int len = m; len <= n; len++) {	            
            res += calcPatterns(-1, len);
            for (int i = 0; i < 9; i++) {	                
                used[i] = false;
            }            
        }
        return res;
    }

    private boolean isValid(int index, int last) {
        if (used[index])
            return false;
        // first digit of the pattern    
        if (last == -1)
            return true;
        // knight moves or adjacent cells (in a row or in a column)	       
        if ((index + last) % 2 == 1)
            return true;
        // indexes are at both end of the diagonals for example 0,0, and 8,8          
        int mid = (index + last)/2;
        if (mid == 4)
            return used[mid];
        // adjacent cells on diagonal  - for example 0,0 and 1,0 or 2,0 and //1,1
        if ((index%3 != last%3) && (index/3 != last/3)) {
            return true;
        }
        // all other cells which are not adjacent
        return used[mid];
    }

    private int calcPatterns(int last, int len) {
        if (len == 0)
            return 1;    
        int sum = 0;
        for (int i = 0; i < 9; i++) {
            if (isValid(i, last)) {
                used[i] = true;
                sum += calcPatterns(i, len - 1);
                used[i] = false;                    
            }
        }
        return sum;
    }
```

#[147. 对链表进行插入排序](https://leetcode-cn.com/problems/insertion-sort-list/) 

>   对链表进行插入排序。
>
>
>   插入排序的动画演示如上。从第一个元素开始，该链表可以被认为已经部分排序（用黑色表示）。
>   每次迭代时，从输入数据中移除一个元素（用红色表示），并原地将其插入到已排好序的链表中。
>
>    
>
>   插入排序算法：
>
>   插入排序是迭代的，每次只移动一个元素，直到所有元素可以形成一个有序的输出列表。
>   每次迭代中，插入排序只从输入数据中移除一个待排序的元素，找到它在序列中适当的位置，并将其插入。
>   重复直到所有输入数据插入完为止。
>
>
>   示例 1：
>
>   输入: 4->2->1->3
>   输出: 1->2->3->4
>   示例 2：
>
>   输入: -1->5->3->4->0
>   输出: -1->0->3->4->5

```java
public ListNode insertionSortList(ListNode head) {
        if(head == null) return head;
        ListNode newHead = new ListNode(0);
        ListNode cur=head, pre=newHead, next;
        while(cur != null){
            next = cur.next;
            if(pre != newHead && pre.val > cur.val)
                pre = newHead;
            while(pre.next != null && pre.next.val < cur.val)
                pre = pre.next;
            cur.next = pre.next;
            pre.next = cur;
            cur = next;
        }
        return newHead.next;
    }
```

#[面试题4 - 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/) 

>   在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
>
>    
>
>   示例:
>
>   现有矩阵 matrix 如下：
>
>   [
>     [1,   4,  7, 11, 15],
>     [2,   5,  8, 12, 19],
>     [3,   6,  9, 16, 22],
>     [10, 13, 14, 17, 24],
>     [18, 21, 23, 26, 30]
>   ]
>   给定 target = 5，返回 true。
>
>   给定 target = 20，返回 false。
>

```java
  public boolean findNumberIn2DArray(int[][] matrix, int target) {
         if(matrix == null || matrix.length == 0) {
            return false;
        }
        int m = matrix.length, n = matrix[0].length;
        //右上角
        int row = 0, col = n - 1;
        while(row < m && col >= 0) {
            //右上角大于target
            if(matrix[row][col] > target) {
                col--;
            }else if(matrix[row][col] < target) {
                row++;
            }else {
                return true;
            }
        }
        return false;
    }
```

#[面试题 01.07 - 旋转矩阵](https://leetcode-cn.com/problems/rotate-matrix-lcci/) 

>   给你一幅由 N × N 矩阵表示的图像，其中每个像素的大小为 4 字节。请你设计一种算法，将图像旋转 90 度。
>
>   不占用额外内存空间能否做到？
>
>    
>
>   示例 1:
>
>   给定 matrix = 
>   [
>     [1,2,3],
>     [4,5,6],
>     [7,8,9]
>   ],
>
>   原地旋转输入矩阵，使其变为:
>   [
>     [7,4,1],
>     [8,5,2],
>     [9,6,3]
>   ]
>   示例 2:
>
>   给定 matrix =
>   [
>     [ 5, 1, 9,11],
>     [ 2, 4, 8,10],
>     [13, 3, 6, 7],
>     [15,14,12,16]
>   ], 
>
>   原地旋转输入矩阵，使其变为:
>   [
>     [15,13, 2, 5],
>     [14, 3, 4, 1],
>     [12, 6, 8, 9],
>     [16, 7,10,11]
>   ]

```java
public void rotate(int[][] matrix) {
        int len1 = matrix.length;
        int len2 = matrix[0].length;
        for(int i = 0; i < len1; i++) {//   \对角线反转
            for(int j = i; j < len2; j++) {
                int t = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = t;
            }
        }
        for(int i = 0; i < len1; i++) {//   左右翻转
            for(int j = 0; j < len2/2; j++) {
                int t = matrix[i][j];
                matrix[i][j] = matrix[i][len1-j-1];
                matrix[i][len1-j-1] = t;
            }
        }
    }
```

#[面试题 08.09 - 括号](https://leetcode-cn.com/problems/bracket-lcci/) 

>   括号。设计一种算法，打印n对括号的所有合法的（例如，开闭一一对应）组合。
>
>   说明：解集不能包含重复的子集。
>
>   例如，给出 n = 3，生成结果为：
>
>   [
>     "((()))",
>     "(()())",
>     "(())()",
>     "()(())",
>     "()()()"
>   ]

```java
private List<String> res;

    public List<String> generateParenthesis(int n) {
        int leftBracket = n, rightBracket = n;
        this.res = new ArrayList<>();
        dfs(leftBracket, rightBracket, new StringBuilder());
        return res;
    }

    private void dfs(int leftBracket, int rightBracket, StringBuilder sb) {
        if (leftBracket == 0 && rightBracket == 0) {
            res.add(sb.toString());
            return;
        }

        if (leftBracket > rightBracket) {
            return;
        }

        if (leftBracket > 0) {
            sb.append("(");
            dfs(leftBracket - 1, rightBracket, sb);
            sb.deleteCharAt(sb.length() - 1);
        }
        if (rightBracket > 0) {
            sb.append(")");
            dfs(leftBracket, rightBracket - 1, sb);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
```

#[剑指 Offer 61 - 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/) 

>   从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。
>
>    
>
>   示例 1:
>
>   输入: [1,2,3,4,5]
>   输出: True
>
>
>   示例 2:
>
>   输入: [0,0,1,2,5]
>   输出: True

```java
public boolean isStraight(int[] nums) {
      int[] cards = new int[14];
        int max = 0,min = 14;

        for(int num : nums) {
            if(num == 0) continue;
            //重复情况判断
            if(cards[num]++ > 0) return false;

            min = Math.min(min,num);
            max = Math.max(max,num);
        }
        return max - min < 5;
    }
```

