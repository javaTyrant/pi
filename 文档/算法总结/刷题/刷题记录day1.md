[TOC]

# [1. 两数之和](https://leetcode-cn.com/problems/two-sum/) 搞定,答应我闭着眼写好吗

> 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
>

```java
    public int[] twoSum(int[] nums, int target) {
      	//为什么索引要做value呢,假设 1,4,2,5 target是3
      	//我们遍历的时候遇到1,因为是第一次,所以我们还缺个2,map中没有二,所以我们
      	//要把1存进去,因为到2的时候,我们要根据3-2把索引的位置取出来,显而易见吧.
        Map<Integer,Integer> map = new HashMap<>();
        for(int i = 0;i < nums.length;i++){
            int com = target - nums[i];
            //注意包含谁
            if(map.containsKey(com)){
                //注意get谁
                return new int[]{map.get(com),i};
            }
                //注意put谁
                map.put(nums[i],i);
        }
        return new int[]{-1,-1};
    }
```



# [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)搞定,滑动窗口很丝滑

> 给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
>
> 示例 1:
>
> 输入: "abcabcbb"
> 输出: 3 
> 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
> 示例 2:
>
> 输入: "bbbbb"
> 输出: 1
> 解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
> 示例 3:
>
> 输入: "pwwkew"
> 输出: 3
> 解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
>      请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。

```java
    //要优雅
    public int lengthOfLongestSubstring(String s) {
        int ans = 0;
        Map<Character,Integer> window = new HashMap<>();
        for(int start = 0,end = 0;end < s.length();end++){
            Character c = s.charAt(end);
          	//如果包含
            if(window.containsKey(c)){
              	//交换start,取较大的start
                start = Math.max(window.get(c),start);
            }
          	//更新长度
            ans = Math.max(ans,end - start + 1);
          	//put c的下一个位置
            window.put(c,end + 1);
        }
        return ans;
    }
```

# [2. 两数相加 ](https://leetcode-cn.com/problems/add-two-numbers/)搞定,太他娘的简单了

>给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。
>
>如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
>
>您可以假设除了数字 0 之外，这两个数都不会以 0 开头。
>
>示例：
>
>输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
>输出：7 -> 0 -> 8
>原因：342 + 465 = 807

```java
   public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        //需要几个指针呢?
        ListNode p = l1, q = l2, cur = dummy;
        int carry = 0;
        while (p != null || q != null) {
            int x = (p == null) ? 0 : p.val;
            int y = (q == null) ? 0 : q.val;
            int sum = carry + x + y;
            carry = sum / 10;
            cur.next = new ListNode(sum % 10);
            cur = cur.next;
            if (p != null) p = p.next;
            if (q != null) q = q.next;
        }
        if (carry > 0) {
            cur.next = new ListNode(carry);
        }
        return dummy.next;
    }
```



# [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/) 中心扩散法搞定,动态规划还要看看

注意多种解法

> 给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。
>
> 示例 1：
>
> 输入: "babad"
> 输出: "bab"
> 注意: "aba" 也是一个有效答案。
> 示例 2：
>
> 输入: "cbbd"
> 输出: "bb"

```java
 public String longestPalindrome(String s) {
        if(s == null || s.length() == 0) return "";
        int start = 0,end = 0;
        for(int i = 0; i < s.length();i++){
            int e1 = expand(s,i,i);
            int e2 = expand(s,i,i+1);
            int len = Math.max(e1,e2);
            if(len>(end - start)){
                start = i - (len-1)/2 ;
                end = i + len/2;
            }
        }
        return s.substring(start,end+1);
    }
    private int expand(String s,int left,int right){
        int l = left,r = right;
        while(l >= 0 && r<s.length() && s.charAt(l) == s.charAt(r)){
            l--;
            r++;
        }
        return r - l - 1;
    }
```



# [42. 接雨水 ](https://leetcode-cn.com/problems/trapping-rain-water/)搞定,方法很多,我先把水干了

> 给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。
>
>  
>
> 示例 1：
>
> 
>
> 输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
> 输出：6
> 解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
> 示例 2：
>
> 输入：height = [4,2,0,3,2,5]
> 输出：9
>
>
> 提示：
>
> n == height.length
> 0 <= n <= 3 * 104
> 0 <= height[i] <= 105

```java
   public int trap(int[] height) {
        //双指针
        int left = 0,right = height.length -1;
        //三个临时变量
        int maxLeft = 0,maxRight = 0,res = 0;
        while(left < right){
            if(height[left] < height[right]){
                if(height[left] >= maxLeft){
                    maxLeft = height[left];
                }else{
                    res += (maxLeft - height[left]);
                }
                left++;
            }else{
                if(height[right] >= maxRight){
                    maxRight = height[right];
                }else{
                    res += (maxRight - height[right]);
                }
                right--;
            }
        }
        return res;
    }
```



# [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/) 搞定,答应我,不要睁着眼睛写

> 反转一个单链表。
>
> 示例:
>
> 输入: 1->2->3->4->5->NULL
> 输出: 5->4->3->2->1->NULL
> 进阶:
> 你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

```java
//迭代 
public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null){
            ListNode temp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
//递归
  public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) return head;
        ListNode p = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return p;
    }
```



# [4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/) 完了,今天忘了这题,干

>给定两个大小为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。请你找出并返回这两个正序数组的中位数。
>
>进阶：你能设计一个时间复杂度为 O(log (m+n)) 的算法解决此问题吗？
>
> 
>
>示例 1：
>
>输入：nums1 = [1,3], nums2 = [2]
>输出：2.00000
>解释：合并数组 = [1,2,3] ，中位数 2
>示例 2：
>
>输入：nums1 = [1,2], nums2 = [3,4]
>输出：2.50000
>解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
>示例 3：
>
>输入：nums1 = [0,0], nums2 = [0,0]
>输出：0.00000
>示例 4：
>
>输入：nums1 = [], nums2 = [1]
>输出：1.00000
>示例 5：
>
>输入：nums1 = [2], nums2 = []
>输出：2.00000

```java
 public double findMedianSortedArrays(int[] A, int[] B) {
        int m = A.length;
        int n = B.length;
        //确保n>=m
        if (m > n) {
            return findMedianSortedArrays(B, A);
        }
        //不加一会死循环
        //两个指针
        int iMin = 0;
        int iMax = m;
        int halfLen = (m + n + 1) / 2;
        while (iMin <= iMax) {
            //i,j两个数组的划分点
            //二分法取中间
            int i = (iMin + iMax) / 2;
            //确保数组的划分
            int j = halfLen - i;
            //j不能等于0
            if (i < iMax && B[j - 1] > A[i]) {
                iMin = i + 1;
                // i is too big
            } else if (i > iMin && A[i - 1] > B[j]) {
                iMax = i - 1;
            } else {
                //i刚刚好
                //记录两个值,左边的最大值,右边的最小值
                int maxLeft;
                //左边的极端值处理
                if (i == 0) {
                    maxLeft = B[j - 1];
                } else if (j == 0) {
                    maxLeft = A[i - 1];
                } else {
                    maxLeft = Math.max(A[i - 1], B[j - 1]);
                }
                //奇数
                if ((m + n) % 2 == 1) {
                    return maxLeft;
                }
                int minRight;
                //右边的极端值处理
                if (i == m) {
                    minRight = B[j];
                } else if (j == n) {
                    minRight = A[i];
                } else {
                    minRight = Math.min(B[j], A[i]);
                }
                return (maxLeft + minRight) / 2.0;
            }
        }
        return 0.0;
  }
```



# [146. LRU缓存机制](https://leetcode-cn.com/problems/lru-cache/) 搞定 其实我就是个链表

>运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制 。
>实现 LRUCache 类：
>
>LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
>int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
>void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。
>
>
>进阶：你是否可以在 O(1) 时间复杂度内完成这两种操作？
>
> 
>
>示例：
>
>输入
>["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
>[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
>输出
>[null, null, null, 1, null, -1, null, -1, 3, 4]
>
>解释
>LRUCache lRUCache = new LRUCache(2);
>lRUCache.put(1, 1); // 缓存是 {1=1}
>lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
>lRUCache.get(1);    // 返回 1
>lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
>lRUCache.get(2);    // 返回 -1 (未找到)
>lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
>lRUCache.get(1);    // 返回 -1 (未找到)
>lRUCache.get(3);    // 返回 3
>lRUCache.get(4);    // 返回 4
>
>
>提示：
>
>1 <= capacity <= 3000
>0 <= key <= 3000
>0 <= value <= 104
>最多调用 3 * 104 次 get 和 put

```java
class LRUCache {
 //内部类 
 class DLinkedNode {
    int key;
    int value;
    DLinkedNode prev;
    DLinkedNode next;
  }
	//
  private void addNode(DLinkedNode node) {
    //处理node
    node.prev = head;
    node.next = head.next;
		//处理head
    head.next.prev = node;
    head.next = node;
  }

  private void removeNode(DLinkedNode node){
		//保存当前节点的首尾节点
    DLinkedNode prev = node.prev;
    DLinkedNode next = node.next;
		//更新
    prev.next = next;
    next.prev = prev;
  }

  private void moveToHead(DLinkedNode node){
 		//先删再增
    removeNode(node);
    addNode(node);
  }

  private DLinkedNode popTail() {
   	//删除尾节点
    DLinkedNode res = tail.prev;
    removeNode(res);
    return res;
  }
	//缓存空间换时间
  private HashMap<Integer, DLinkedNode> cache =
          new HashMap<Integer, DLinkedNode>();
  //存了多少
  private int size;
  //上限
  private int capacity;
  //head tail
  private DLinkedNode head, tail;

  public LRUCache(int capacity) {
    this.size = 0;
    this.capacity = capacity;

    head = new DLinkedNode();

    tail = new DLinkedNode();
		//首尾相连
    head.next = tail;
    tail.prev = head;
  }
	//
  public int get(int key) {
    //先从缓存取
    DLinkedNode node = cache.get(key);
    if (node == null) return -1;
    //移到head后
    moveToHead(node);
		//返回value
    return node.value;
  }

  public void put(int key, int value) {
    //先从缓存取
    DLinkedNode node = cache.get(key);
		//新节点
    if(node == null) {
      //构造node
      DLinkedNode newNode = new DLinkedNode();
      newNode.key = key;
      newNode.value = value;
			//放入缓存
      cache.put(key, newNode);
      //添加节点
      addNode(newNode);
			//尺码++
      ++size;
			//如果超过上限了
      if(size > capacity) {
        // pop the tail
        //删除tail
        DLinkedNode tail = popTail();
        //缓存移除
        cache.remove(tail.key);
        --size;
      }
    } else {
      //老节点更新value,moveToHead
      node.value = value;
      moveToHead(node);
    }
  }
}
```



# [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)搞定,分治法还要再看看

> 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> 示例:
>
> 输入: [-2,1,-3,4,-1,2,1,-5,4]
> 输出: 6
> 解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
> 进阶:
>
> 如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。
>

```java
 public int maxSubArray(int[] nums) {
        if (nums.length == 0) return 0;
        int res = Integer.MIN_VALUE;
        int maxsum = 0;
        for (int num : nums) {
            maxsum = Math.max(0, maxsum) + num;
            res = Math.max(res, maxsum);
        }
        return res;
    }
```



# [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)搞定

>
> 将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 
>
>  
>
> **示例：**
>
> ```java
> 输入：1->2->4, 1->3->4
> 输出：1->1->2->3->4->4
> ```

```java
  public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode res = new ListNode(-1);
        ListNode cur = res;
        while(l1 != null && l2 != null){
            if(l1.val < l2.val){
                cur.next = l1;
                l1 = l1.next;
            }else{
                cur.next = l2;
                l2 = l2.next;
            }
            cur = cur.next;
        }
        cur.next = l1 == null ? l2 : l1;
        return res.next;
    }
```



# [15. 三数之和](https://leetcode-cn.com/problems/3sum/)搞定 编程内功你要不要

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
>
> 注意：答案中不可以包含重复的三元组。
>
>  
>
> 示例：
>
> 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
>
> 满足要求的三元组集合为：
> [
>   [-1, 0, 1],
>   [-1, -1, 2]
> ]

本题最重要的一步就是去重

```java
 public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        //基操排序
        Arrays.sort(nums);
        int length = nums.length;
        //基操for循环
        for (int i = 0; i < length; ++i) {
            if (nums[i] > 0) return result;
            //去重
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            int cur = nums[i];
            //基操双指针
            int left = i + 1, right = length - 1;
            //基操while
            while (left < right) {
                int sum = cur + nums[left] + nums[right];
                if (sum == 0) {
                    List<Integer> l1 = new ArrayList<>();
                    l1.add(cur);
                    l1.add(nums[left]);
                    l1.add(nums[right]);
                    result.add(l1);
                    //再去重
                    while (left < right && nums[left + 1] == nums[left]) {
                        ++left;
                    }
                    while (left < right && nums[right - 1] == nums[right]) {
                        --right;
                    }
                    //基操移动指针
                    ++left;
                    --right;

                } else if (sum < 0) {
                    //两种情况
                    ++left;
                } else {
                    --right;
                }
            }
        }
        return result;
    }
```



# [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/) 细节太多了,炸裂

>
>
>给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。
>
>k 是一个正整数，它的值小于或等于链表的长度。
>
>如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序。
>
> 
>
>示例：
>
>给你这个链表：1->2->3->4->5
>
>当 k = 2 时，应当返回: 2->1->4->3->5
>
>当 k = 3 时，应当返回: 3->2->1->4->5
>
> 
>
>说明：
>
>你的算法只能使用常数的额外空间。
>你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

```java
public ListNode reverseKGroup(ListNode head, int k) {
        ListNode preLast = null, curHead = new ListNode(-1), p = head, q, tmp;
        head = null;
        while(p != null){
            curHead.next = null;
            int i = 0;
            tmp = p;
            while(p != null && i < k){
                q = p.next;
                p.next = curHead.next;
                curHead.next = p;
                p = q;
                i++;
            }
            if(i != k){
                ListNode pp = curHead.next;
                curHead.next = null;
                while(pp != null){
                    q = pp.next;
                    pp.next = curHead.next;
                    curHead.next = pp;
                    pp = q;
                }
            }
            if(preLast != null){
                preLast.next = curHead.next;  
            }
            preLast = tmp;
            if(head == null){
                head = curHead.next;

            }
        }
        return head;
    }
```

better code

```java
public ListNode reverseKGroup(ListNode head, int k) {
        ListNode hair = new ListNode(0);
        hair.next = head;
        ListNode pre = hair;

        while (head != null) {
            ListNode tail = pre;
            // 查看剩余部分长度是否大于等于 k
            for (int i = 0; i < k; ++i) {
                tail = tail.next;
                if (tail == null) {
                    return hair.next;
                }
            }
            ListNode nex = tail.next;
            ListNode[] reverse = myReverse(head, tail);
            head = reverse[0];
            tail = reverse[1];
            // 把子链表重新接回原链表
            pre.next = head;
            tail.next = nex;
            pre = tail;
            head = tail.next;
        }

        return hair.next;
    }

    public ListNode[] myReverse(ListNode head, ListNode tail) {
        ListNode prev = tail.next;
        ListNode p = head;
        while (prev != tail) {
            ListNode nex = p.next;
            p.next = prev;
            prev = p;
            p = nex;
        }
        return new ListNode[]{tail, head};
    }
```



# [11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/) 搞定,双指针这么简单

>  给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
>
> 说明：你不能倾斜容器。
>
> ![img](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)
>
> 输入：[1,8,6,2,5,4,8,3,7]
> 输出：49 
> 解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
> 示例 2：
>
> 输入：height = [1,1]
> 输出：1
> 示例 3：
>
> 输入：height = [4,3,2,1,4]
> 输出：16
> 示例 4：
>
> 输入：height = [1,2,1]
> 输出：2

```java
public int maxArea(int[] height) {
        int left = 0;
        int right = height.length -1;
        int res = 0;
        if(height == null||height.length == 0){
            return res;
        }
        while(left < right){
          	//取出短板
            int h = Math.min(height[left],height[right]);
          	//如果大于则更新
            res = Math.max(res,(right-left)*h);
          	//否则开始双指针移动
            if(height[left] < height[right]){
                left++;
            }else{
                right--;
            }
        }
        return res;
    }

```



# [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/) 搞定,柳暗花明又一村

>  给你一个升序排列的整数数组 nums ，和一个整数 target 。
>
> 假设按照升序排序的数组在预先未知的某个点上进行了旋转。（例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] ）。
>
> 请你在数组中搜索 target ，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。
>
>
> 示例 1：
>
> 输入：nums = [4,5,6,7,0,1,2], target = 0
> 输出：4
> 示例 2：
>
> 输入：nums = [4,5,6,7,0,1,2], target = 3
> 输出：-1
> 示例 3：
>
> 输入：nums = [1], target = 0
> 输出：-1

```java
  public static int search(int[] nums, int target) {
        int len = nums.length;
        //双指针
        int left = 0, right = len - 1;
        //
        while (left <= right) {
            //取中间
            int mid = (left + right) / 2;
            if (nums[mid] == target) {
                return mid;
                //中间小于右边 ,说明mid右边是有序的
            } else if (nums[mid] < nums[right]) {
                //mid < target 且 target < 右边
                //说明target在右边,所以要改变left的值
                if (nums[mid] < target && target <= nums[right])
                    left = mid + 1;
                else
                    //target在左边
                    right = mid - 1;
                //中间大于右边,说明左边是有序的
            } else {
                //target在左边,要改变right的值
                if (nums[left] <= target && target < nums[mid])
                    right = mid - 1;
                else
                    left = mid + 1;
            }
        }
        return -1;
    
```



# [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/) 搞定 这个系列不简单

>  给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
>
> 如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。
>
> 注意：你不能在买入股票前卖出股票。
>
>  
>
> 示例 1:
>
> 输入: [7,1,5,3,6,4]
> 输出: 5
> 解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
>      注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
> 示例 2:
>
> 输入: [7,6,4,3,1]
> 输出: 0
> 解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

这个和最大子序和也有相通之处

思路:保存遇到的最小值,循环数组,用每个值减去最小值,保存最大的值

所以需要两个变量,一个保存最小值,一个保存最大的差值.

```java
  public int maxProfit(int[] prices) {
        int res = 0;
        int low = Integer.MAX_VALUE;
        for(int price : prices){
            //保存买入最小
            low = Math.min(price,low);
            //保存差值最大
            res = Math.max(res,price - low);
        }
        return res;
    }
```



# [23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/) 好多指针

>  给你一个链表数组，每个链表都已经按升序排列。
>
> 请你将所有链表合并到一个升序链表中，返回合并后的链表。
>
>  
>
> 示例 1：
>
> 输入：lists = [[1,4,5],[1,3,4],[2,6]]
> 输出：[1,1,2,3,4,4,5,6]
> 解释：链表数组如下：
> [
>   1->4->5,
>   1->3->4,
>   2->6
> ]
> 将它们合并到一个有序链表中得到。
> 1->1->2->3->4->4->5->6
> 示例 2：
>
> 输入：lists = []
> 输出：[]
> 示例 3：
>
> 输入：lists = [[]]
> 输出：[]

注意多种解决思路.

```java
public ListNode mergeKLists(ListNode[] lists) {
        int interval = 1;
        while (interval < lists.length) {
            for (int i = 0; i < lists.length - interval; i += interval * 2) {
                lists[i] = merge2Lists(lists[i], lists[i + interval]);
            }
            interval *= 2;
        }
        return lists.length > 0 ? lists[0] : null;
    }

    public ListNode merge2Lists(ListNode l1, ListNode l2) {
        if (l1 == null)
            return l2;
        if (l2 == null)
            return l1;
        if (l1.val < l2.val) {
            l1.next = merge2Lists(l1.next, l2);
            return l1;
        } else {
            l2.next = merge2Lists(l1, l2.next);
            return l2;
        }
    }
```



# [56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/) 搞定,其实很简单

>给出一个区间的集合，请合并所有重叠的区间。
>
> 
>
>示例 1:
>
>输入: intervals = [[1,3],[2,6],[8,10],[15,18]]
>输出: [[1,6],[8,10],[15,18]]
>解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
>示例 2:
>
>输入: intervals = [[1,4],[4,5]]
>输出: [[1,5]]
>解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。 

- 时间复杂度：$$O(nlog n)O(nlogn)$$，其中 nn 为区间的数量。除去排序的开销，我们只需要一次线性扫描，所以主要的时间开销是排序的 O(n\log n)O(nlogn)。

- 空间复杂度：$$O(log n)O(logn)$$，其中 nn 为区间的数量。这里计算的是存储答案之外，使用的额外空间。O(\log n)O(logn) 即为排序所需要的空间复杂度。

```java
  public int[][] merge(int[][] intervals) {
        // 先按照区间起始位置排序
        Arrays.sort(intervals, Comparator.comparingInt(v -> v[0]));
        // 遍历区间
        int[][] res = new int[intervals.length][2];
        int idx = -1;
        for (int[] interval : intervals) {
            // 如果结果数组是空的，或者当前区间的起始位置 > 结果数组中最后区间的终止位置，
            // 则不合并，直接将当前区间加入结果数组。
          	//res[idx][1]
            if (idx == -1 || interval[0] > res[idx][1]) {
                res[++idx] = interval;
            } else {
                // 反之将当前区间合并至结果数组的最后区间
                res[idx][1] = Math.max(res[idx][1], interval[1]);
            }
        }
        return Arrays.copyOf(res, idx + 1);
    }
```



# [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)搞定

>  给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
>
> 有效字符串需满足：
>
> 左括号必须用相同类型的右括号闭合。
> 左括号必须以正确的顺序闭合。
> 注意空字符串可被认为是有效字符串。

```java
   public boolean isValid(String s) {
        int len = 0;
        char[] arr = new char[s.length()];
        for(int i = 0;i < s.length();i++){
            char c = s.charAt(i);
            if(c == '[' || c == '(' || c == '{'){
                arr[len] = c;
                len++;
            }else if(len > 0){
                if(c == ')' && arr[len - 1] != '('){
                    return false;
                }
                if(c == ']' && arr[len - 1] != '['){
                   return false;
                }
                if(c == '}' && arr[len - 1] != '{'){
                    return false;
                }
                len--;
            }else{
                return false;
            }
        }
        return len == 0;                             
    }
```



# [46. 全排列](https://leetcode-cn.com/problems/permutations/) 搞定,经典回溯,所有关键词

> 给定一个 没有重复 数字的序列，返回其所有可能的全排列。
>
> 示例:
>
> 输入: [1,2,3]
> 输出:
> [
>   [1,2,3],
>   [1,3,2],
>   [2,1,3],
>   [2,3,1],
>   [3,1,2],
>   [3,2,1]
> ]

```java
  public List<List<Integer>> permute(int[] nums) {
        List<Integer> list = new ArrayList<>();
        for(int num : nums){
            list.add(num);
        }
        List<List<Integer>> res = new ArrayList<>();
        findPermutes(list,res,0,new ArrayList<>());
        return res;
    }
    public void findPermutes(List<Integer> nums,List<List<Integer>> res,
                             int index,List<Integer> current){
        if(index == nums.size()){
            res.add(new ArrayList<>(nums));
        }else{
            for(int i = index;i < nums.size();i++){
                Collections.swap(nums,i,index);
                findPermutes(nums,res,index+1,current);
                Collections.swap(nums,i,index);
            }
        }
    }
```



# [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)搞定 其实我就是个斐波拉契

> 假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
>
> 每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> 注意：给定 n 是一个正整数。
>
> 示例 1：
>
> 输入： 2
> 输出： 2
> 解释： 有两种方法可以爬到楼顶。
>
> 1.  1 阶 + 1 阶
> 2.  2 阶
> 示例 2：
>
> 输入： 3
> 输出： 3
> 解释： 有三种方法可以爬到楼顶。
> 1.  1 阶 + 1 阶 + 1 阶
> 2.  1 阶 + 2 阶
> 3.  2 阶 + 1 阶
>

如果优化递归?

```java
  public int climbStairs(int n) {
        int[] memo = {1,1};
        for(int i = 1;i < n;i++){
            int temp = memo[1];
            memo[1] += memo[0];
            memo[0] = temp;
        }
        return memo[1];
    }
```

总结:今天工具箱里多了哪几种方法呢?