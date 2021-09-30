[toc]

#[349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

>   给定两个数组，编写一个函数来计算它们的交集。
>
>    
>
>   示例 1：
>
>   输入：nums1 = [1,2,2,1], nums2 = [2,2]
>   输出：[2]
>   示例 2：
>
>   输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
>   输出：[9,4]
>
>
>   说明：
>
>   输出结果中的每个元素一定是唯一的。
>   我们可以不考虑输出结果的顺序。

```java
 public int[] intersection(int[] nums1, int[] nums2) {
  			int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        //找到num1的最大值与最小值
        for (int i = 0; i < nums1.length; i++) {
            max = max < nums1[i] ? nums1[i] : max;
            min = min > nums1[i] ? nums1[i] : min;
        }
        boolean[] booleanArray = new boolean[max - min + 1];
        for (int i = 0; i < nums1.length; i++) {
            //为什么要减min?
            booleanArray[nums1[i] - min] = true;
        }
        int index = 0;
        int[] tempArray = new int[nums2.length];
        for (int i = 0; i < nums2.length; i++) {
            if (!(nums2[i] < min) && !(nums2[i] > max) && booleanArray[nums2[i] - min]) {
                booleanArray[nums2[i] - min] = false;
                tempArray[index++] = nums2[i];
            }
        }
        return Arrays.copyOf(tempArray, index);
    }
```

# [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

>   给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。
>
>   返回滑动窗口中的最大值。
>
>    
>
>   进阶：
>
>   你能在线性时间复杂度内解决此题吗？
>
>    
>
>   示例:
>
>   输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
>   输出: [3,3,5,5,6,7] 
>   解释: 
>
>     滑动窗口的位置                最大值
>   ---------------               -----
>   [1  3  -1] -3  5  3  6  7       3
>    1 [3  -1  -3] 5  3  6  7       3
>    1  3 [-1  -3  5] 3  6  7       5
>    1  3  -1 [-3  5  3] 6  7       5
>    1  3  -1  -3 [5  3  6] 7       6
>    1  3  -1  -3  5 [3  6  7]      7

```java
public int[] maxSlidingWindow(int[] nums, int k) {
        //3 1 2 4 3 3  k = 3
        if(nums.length == 0 || k == 0) return new int[0];
        int resLen = nums.length - k + 1;
        int[] res = new int[resLen];
        int max = Integer.MIN_VALUE;
        int p = -1;
        for(int i = 0;i < k;i++){
            if(nums[i] > max){
                max = nums[i];
                p = i;
            }
        }
        res[0] = max;
        int start = 0;
        int end = k - 1;
        // 1 2 1 3 注意边界值
        for(int i = 1;i < res.length;i++){
            start++;
            end++;
            //
            if(p >= start){
                if(nums[p]< nums[end]){
                    p = end;
                    res[i] = nums[end];
                }else{
                   res[i] = nums[p];
                }
            }else{
                if(nums[end] >= nums[p]){
                    p = end;
                    res[i] = nums[end];
                }else{
                    p = end;
                    res[i] = nums[end];
                    for(int m = start;m < end;m++){
                        if(nums[m] > res[i]){
                            res[i] = nums[m];
                            p = m;
                        }
                    }
                }
            }
        }
        return res;
    }
```

# [402. 移掉K位数字](https://leetcode-cn.com/problems/remove-k-digits/)

>   给定一个以字符串表示的非负整数 num，移除这个数中的 k 位数字，使得剩下的数字最小。
>
>   注意:
>
>   num 的长度小于 10002 且 ≥ k。
>   num 不会包含任何前导零。
>   示例 1 :
>
>   输入: num = "1432219", k = 3
>   输出: "1219"
>   解释: 移除掉三个数字 4, 3, 和 2 形成一个新的最小的数字 1219。
>   示例 2 :
>
>   输入: num = "10200", k = 1
>   输出: "200"
>   解释: 移掉首位的 1 剩下的数字为 200. 注意输出不能有任何前导零。
>   示例 3 :
>
>   输入: num = "10", k = 2
>   输出: "0"
>   解释: 从原数字移除所有的数字，剩余为空就是0。

```java
      if (num.length() == k) return "0";
        StringBuilder s = new StringBuilder(num);
        for (int i = 0; i < k; i++) {
            int idx = 0;
            for (int j = 1; j < s.length() && s.charAt(j) >= s.charAt(j - 1); j++) idx = j;
            s.delete(idx, idx + 1);
            while (s.length() > 1 && s.charAt(0) == '0') s.delete(0, 1);
        }
        return s.toString();
    }
```



#[剑指 Offer 51 - 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

>   在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。
>
>    
>
>   示例 1:
>
>   输入: [7,5,6,4]
>   输出: 5

```java
  public int reversePairs(int[] nums) {
        if(nums==null||nums.length<1)
            return 0;
        int[] copy=new int[nums.length];
        System.arraycopy(nums,0,copy,0,copy.length);

        return mergeSort(nums,copy,0,nums.length-1);
    }

     public int mergeSort(int[] nums,int[] copy,int start,int end) {
        if(start==end){
            copy[start]=nums[start];
            return 0;
        }
        int middle=(start+end)>>1;
        int leftcount=mergeSort(copy,nums,start,middle);
        int rightcount=mergeSort(copy,nums,middle+1,end);
        int count=0;
        int i=middle,j=end;
        int lastindex=end;
        while(i>=start&&j>=middle+1){
            if(nums[i]>nums[j]){
                copy[lastindex--]=nums[i--];
                count+=j-middle;
            }
            else
                copy[lastindex--]=nums[j--];
        }
        while(i>=start)
            copy[lastindex--]=nums[i--];
        while(j>=middle+1) 
            copy[lastindex--]=nums[j--];

        return count+leftcount+rightcount;
    }
```



# [16. 最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

>   给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。
>
>   示例：
>
>   输入：nums = [-1,2,1,-4], target = 1
>   输出：2
>   解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2) 。

```java
 public int threeSumClosest(int[] nums, int target) {
   			//保存最小值
        int closest = Integer.MAX_VALUE;
        int res = 0;
        Arrays.sort(nums);
        for(int i = 0;i< nums.length - 2;i++){
            int l = i + 1;
            int r = nums.length - 1;
            while(l < r){
                int sum = nums[i] + nums[l] + nums[r];
              	//求出
                int min = sum - target;
                if(min > 0){
                    r--;
                }else if(min < 0){
                    l++;
                }else{
                    return sum;
                }
                if(Math.abs(min) < closest){
                    closest = Math.abs(min);
                    res = sum;
                }
            }
        }
        return res;
    }
```

# [203. 移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements/)

>   删除链表中等于给定值 ***val\*** 的所有节点。
>
>   **示例:**
>
>   ```java
>   输入: 1->2->6->3->4->5->6, val = 6
>   输出: 1->2->3->4->5
>   ```

```java
   public ListNode removeElements(ListNode head, int val) {
     		//需要一个dummy,一个指针p,dummy赋值
        ListNode dummyHead = new ListNode(0), p = dummyHead;
        dummyHead.next = head;
        while (p.next != null)
            if (p.next.val == val)
              p.next = p.next.next; 
            else p = p.next;
        return dummyHead.next;
    }
```

# [28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

>   
>   实现 [strStr()](https://baike.baidu.com/item/strstr/811469) 函数。
>
>   给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回 **-1**。
>
>   **示例 1:**
>
>   ```
>   输入: haystack = "hello", needle = "ll"
>   输出: 2
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: haystack = "aaaaa", needle = "bba"
>   输出: -1
>   ```

```java
  public int strStr(String haystack, String needle) {
        int L = needle.length(), n = haystack.length();
        if (L == 0) return 0;

        int pn = 0;
        while (pn < n - L + 1) {
            // find the position of the first needle character
            // in the haystack string
            while (pn < n - L + 1 && haystack.charAt(pn) != needle.charAt(0)) ++pn;

            // compute the max match string
            int currLen = 0, pL = 0;
            while (pL < L && pn < n && haystack.charAt(pn) == needle.charAt(pL)) {
                ++pn;
                ++pL;
                ++currLen;
            }

            // if the whole needle string is found,
            // return its start position
            if (currLen == L) return pn - L;

            // otherwise, backtrack
            pn = pn - currLen + 1;
        }
        return -1;
    }
```

# [283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

>   给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
>
>   示例:
>
>   输入: [0,1,0,3,12]
>   输出: [1,3,12,0,0]
>   说明:
>
>   必须在原数组上操作，不能拷贝额外的数组。
>   尽量减少操作次数。

```java
 public void moveZeroes(int[] nums) {
    if (nums.length < 2) {
            return;
        }
        int ps = 0;
        while (ps < nums.length) {
            while (ps < nums.length && nums[ps] != 0) {
                ps++;
            }
            int pe = ps + 1;
            while (pe < nums.length - 1 && nums[pe] == 0) {
                pe++;
            }
            if (pe == nums.length || ps == nums.length) {
                break;
            }
            int temp = nums[ps];
            nums[ps] = nums[pe];
            nums[pe] = temp;
            if (ps < nums.length) {
                ps++;
            } else {
                break;
            }
        }
    }
```

#[剑指 Offer 11 - 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

>   把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  
>
>   示例 1：
>
>   输入：[3,4,5,1,2]
>   输出：1
>   示例 2：
>
>   输入：[2,2,2,0,1]
>   输出：0

```java
  public int minArray(int[] numbers) {
        int l = 0,r = numbers.length - 1;
        while(l < r){
            int mid = (r - l)/2 + l;
            if(numbers[r] > numbers[mid]){
                r = mid;
            }else if(numbers[r] < numbers[mid]){
                //mid处可以放弃的
                l = mid + 1;
            }else r--;
        }
        return numbers[l];
    }
```

# [177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

>   编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。
>
>   +----+--------+
>   | Id | Salary |
>   +----+--------+
>   | 1  | 100    |
>   | 2  | 200    |
>   | 3  | 300    |
>   +----+--------+
>   例如上述 Employee 表，n = 2 时，应返回第二高的薪水 200。如果不存在第 n 高的薪水，那么查询应返回 null。
>
>   +------------------------+
>   | getNthHighestSalary(2) |
>   +------------------------+
>   | 200                    |
>   +------------------------+

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    set n = n-1;
  RETURN (
      # Write your MySQL query statement below.
     select distinct Salary
      from Employee order by Salary desc
      limit 1 offset n
  );
END
```

#[44. 通配符匹配](https://leetcode-cn.com/problems/wildcard-matching/)

>   给定一个字符串 (s) 和一个字符模式 (p) ，实现一个支持 '?' 和 '*' 的通配符匹配。
>
>   '?' 可以匹配任何单个字符。
>   '*' 可以匹配任意字符串（包括空字符串）。
>   两个字符串完全匹配才算匹配成功。
>
>   说明:
>
>   s 可能为空，且只包含从 a-z 的小写字母。
>   p 可能为空，且只包含从 a-z 的小写字母，以及字符 ? 和 *。
>   示例 1:
>
>   输入:
>   s = "aa"
>   p = "a"
>   输出: false
>   解释: "a" 无法匹配 "aa" 整个字符串。
>   示例 2:
>
>   输入:
>   s = "aa"
>   p = "*"
>   输出: true
>   解释: '*' 可以匹配任意字符串。
>   示例 3:
>
>   输入:
>   s = "cb"
>   p = "?a"
>   输出: false
>   解释: '?' 可以匹配 'c', 但第二个 'a' 无法匹配 'b'。
>   示例 4:
>
>   输入:
>   s = "adceb"
>   p = "*a*b"
>   输出: true
>   解释: 第一个 '*' 可以匹配空字符串, 第二个 '*' 可以匹配字符串 "dce".
>   示例 5:
>
>   输入:
>   s = "acdcb"
>   p = "a*c?b"
>   输出: false

```java
 public boolean isMatch(String s, String p) {
        boolean[][] value = new boolean[p.length()+1][s.length()+1];
        value[0][0] = true;
        for(int i = 1;i <= s.length();i++){
            value[0][i] = false;
        }
        for(int i = 1;i <= p.length(); i++){
            if(p.charAt(i-1) == '*'){
                value[i][0] = value[i-1][0];
                for(int j = 1;j <= s.length(); j++){
                    value[i][j] = (value[i][j-1] || value[i-1][j]);
                }
            }else if(p.charAt(i-1) == '?'){
                value[i][0] = false;
                for(int j = 1;j <= s.length(); j++){
                    value[i][j] = value[i-1][j-1];
                }
            }else {
                value[i][0] = false;
                for(int j = 1;j <= s.length(); j++){
                    value[i][j] = s.charAt(j-1) == p.charAt(i-1) && value[i-1][j-1];
                }
            }

        }
        return value[p.length()][s.length()];
    }
```

# [354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

>   给定一些标记了宽度和高度的信封，宽度和高度以整数对形式 (w, h) 出现。当另一个信封的宽度和高度都比这个信封大的时候，这个信封就可以放进另一个信封里，如同俄罗斯套娃一样。
>
>   请计算最多能有多少个信封能组成一组“俄罗斯套娃”信封（即可以把一个信封放到另一个信封里面）。
>
>   说明:
>   不允许旋转信封。
>
>   示例:
>
>   输入: envelopes = [[5,4],[6,4],[6,7],[2,3]]
>   输出: 3 
>   解释: 最多信封的个数为 3, 组合为: [2,3] => [5,4] => [6,7]。

```java
public int maxEnvelopes(int[][] envelopes) {
        int n = envelopes.length;
        Arrays.sort(envelopes, (i1, i2) -> {
            if (i1[0] != i2[0]) {
                return i1[0] - i2[0];
            } else {
                return i1[1] - i2[1];
            }
        });
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (envelopes[j][0] < envelopes[i][0] && 
                    envelopes[j][1] < envelopes[i][1]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
        }
        // 当输入为空时，dp 长度为 0，使用 Optinal 类的 orElse 方法
        return Arrays.stream(dp).max().orElse(0);
    }
```

# [剑指 Offer 10- II - 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

>   一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。
>
>   答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。
>
>   示例 1：
>
>   输入：n = 2
>   输出：2
>   示例 2：
>
>   输入：n = 7
>   输出：21
>   示例 3：
>
>   输入：n = 0
>   输出：1
>   提示：
>
>   0 <= n <= 100
>

```java
  public int numWays(int n) {
        int a = 0;
        int b = 1;
        for(int i=1;i<=n;i++){
            int temp = b;
            b = (a+b)%1000000007;
            a = (temp)%1000000007;
        }
        int res =   b%1000000007;
        return res;
    }
```

# [剑指 Offer 38 - 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

>   
>   输入一个字符串，打印出该字符串中字符的所有排列。
>
>    
>
>   你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。
>
>    
>
>   **示例:**
>
>   ```
>   输入：s = "abc"
>   输出：["abc","acb","bac","bca","cab","cba"]
>   ```

```java
private List<String> list = new ArrayList<>();

    public String[] permutation(String s) {
        perm(s.toCharArray(), 0, s.length() - 1);
        
        return list.toArray(new String[0]);
    }

    private void perm(char[] seq, int curPos, int n) {
        if (curPos == n) {
            list.add(new String(seq));
        } else {
            for (int i = curPos; i <= n; i++) {
                // 判断第i个元素是否在seq[curPos,i－1]中出现过,如果出现过就不用交换了
                if(!findSame(seq,curPos,i)){
                    swap(seq, curPos, i);
                    perm(seq, curPos + 1, n);
                    swap(seq, i, curPos);
                }
            }
        }
    }

    private boolean findSame(char[] seq, int from, int candidate) {
        for (int j = from; j < candidate; j++) {
            if(seq[j] == seq[candidate]){
                return true;
            }
        }
        return false;
    }
```

# [424. 替换后的最长重复字符](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)

>   给你一个仅由大写英文字母组成的字符串，你可以将任意位置上的字符替换成另外的字符，总共可最多替换 k 次。在执行上述操作后，找到包含重复字母的最长子串的长度。
>
>   注意:
>   字符串长度 和 k 不会超过 104。
>
>   示例 1:
>
>   输入:
>   s = "ABAB", k = 2
>
>   输出:
>   4
>
>   解释:
>   用两个'A'替换为两个'B',反之亦然。
>   示例 2:
>
>   输入:
>   s = "AABABBA", k = 1
>
>   输出:
>   4
>
>   解释:
>   将中间的一个'A'替换为'B',字符串变为 "AABBBBA"。
>   子串 "BBBB" 有最长重复字母, 答案为 4。

```java
 private int[] map = new int[26];
    public int characterReplacement(String s, int k) {
        if (s == null) {
            return 0;
        }
        char[] chars = s.toCharArray();
        int left = 0;
        int right = 0;
        int historyCharMax = 0;
        for (right = 0; right < chars.length; right++) {
            int index = chars[right] - 'A';
            map[index]++;
            historyCharMax = Math.max(historyCharMax, map[index]);
            if (right - left + 1 > historyCharMax + k) {
                map[chars[left] - 'A']--;
                left++;
            }
        }
        return chars.length - left;
    }
```

# [189. 旋转数组](https://leetcode-cn.com/problems/rotate-array/)

>   给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。
>
>   示例 1:
>
>   输入: [1,2,3,4,5,6,7] 和 k = 3
>   输出: [5,6,7,1,2,3,4]
>   解释:
>   向右旋转 1 步: [7,1,2,3,4,5,6]
>   向右旋转 2 步: [6,7,1,2,3,4,5]
>   向右旋转 3 步: [5,6,7,1,2,3,4]
>   示例 2:
>
>   输入: [-1,-100,3,99] 和 k = 2
>   输出: [3,99,-1,-100]
>   解释: 
>   向右旋转 1 步: [99,-1,-100,3]
>   向右旋转 2 步: [3,99,-1,-100]
>   说明:
>
>   尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。
>   要求使用空间复杂度为 O(1) 的 原地 算法。

```java
    public void rotate(int[] nums, int k) {
      	int[] aux = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            aux[i] = nums[i];
        }
        for (int j = 1; j <= k; j++) {
            int back = aux[nums.length - 1];
            for (int i = 1; i < nums.length; i++) {
                nums[i] = aux[i - 1];
            }
            nums[0] = back;
            for (int i = 0; i < nums.length; i++) {
                aux[i] = nums[i];
            }
        }
    }
```

# [470. 用 Rand7() 实现 Rand10()](https://leetcode-cn.com/problems/implement-rand10-using-rand7/)

>   已有方法 rand7 可生成 1 到 7 范围内的均匀随机整数，试写一个方法 rand10 生成 1 到 10 范围内的均匀随机整数。
>
>   不要使用系统的 Math.random() 方法。
>

```java
 public int rand10() {
          while(true){
        int num = (rand7()-1)*7 + rand7()-1;
        if(num < 40)return num%10+1;
        }
    }
```

# [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

>   
>   给定一个数组，它的第 *i* 个元素是一支给定的股票在第 *i* 天的价格。
>
>   设计一个算法来计算你所能获取的最大利润。你最多可以完成 *两笔* 交易。
>
>   **注意:** 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
>
>   **示例 1:**
>
>   ```
>   输入: [3,3,5,0,0,3,1,4]
>   输出: 6
>   解释: 在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
>        随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
>   ```
>
>   **示例 2:**
>
>   ```
>   输入: [1,2,3,4,5]
>   输出: 4
>   解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。   
>        注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。   
>        因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
>   ```
>
>   **示例 3:**
>
>   ```
>   输入: [7,6,4,3,1] 
>   输出: 0 
>   解释: 在这个情况下, 没有交易完成, 所以最大利润为 0。
>   ```

```java
public int maxProfit(int[] prices) {
        int fstBuy = Integer.MIN_VALUE, fstSell = 0;
        int secBuy = Integer.MIN_VALUE, secSell = 0;
        for(int p : prices) {
            fstBuy = Math.max(fstBuy, -p);
            fstSell = Math.max(fstSell, fstBuy + p);
            secBuy = Math.max(secBuy, fstSell - p);
            secSell = Math.max(secSell, secBuy + p); 
        }
        return secSell;
    }
```

