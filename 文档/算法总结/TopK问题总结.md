## x基操:自己实现一个优先队列



## 快排的变形



## PriorityQueue

底层结构

```java
transient Object[] queue
```

## [347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

最简单的做法是给「出现次数数组」排序。但由于可能有 $$O(N)$$ 个不同的出现次数（其中 N 为原数组长度），故总的算法复杂度会达到 $$O(NlogN)$$，不满足题目的要求。

在这里，我们可以利用堆的思想：建立一个小顶堆，然后遍历「出现次数数组」：

- 如果堆的元素个数小于 k，就可以直接插入堆中。
- 如果堆的元素个数等于 k，则检查堆顶与当前出现次数的大小。如果堆顶更大，说明至少有 k 个数字的出现次数比当前值大，故舍弃当前值；否则，就弹出堆顶，并将当前值插入堆中。

遍历完成后，堆中的元素就代表了「出现次数数组」中前 k 大的值。

```java
public int[] topKFrequent(int[] nums, int k) {
  			//统计出现的次数
        Map<Integer, Integer> occurrences = new HashMap<Integer, Integer>();
        for (int num : nums) {
            occurrences.put(num, occurrences.getOrDefault(num, 0) + 1);
        }

        // int[] 的第一个元素代表数组的值，第二个元素代表了该值出现的次数
       	PriorityQueue<int[]> queue = new PriorityQueue<>(Comparator.comparingInt(m -> m[1]));
        for (Map.Entry<Integer, Integer> entry : occurrences.entrySet()) {
          	//num是值,count是出现的次数
            int num = entry.getKey(), count = entry.getValue();
          	//如果堆元素的个数等于k,插入堆中
            if (queue.size() == k) {
                if (queue.peek()[1] < count) {
                  	//删除旧值
                    queue.poll();
                  	//插入新值
                    queue.offer(new int[]{num, count});
                }
            } else {
              	//小于直接插入
                queue.offer(new int[]{num, count});
            }
        }
  			//统计个数
        int[] ret = new int[k];
        for (int i = 0; i < k; ++i) {
            ret[i] = queue.poll()[0];
        }
        return ret;
    }
```

#### 方法二：基于快速排序

![image-20201205113346449](/Users/lumac/Library/Application Support/typora-user-images/image-20201205113346449.png)

```java
public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> occurrences = new HashMap<Integer, Integer>();
        for (int num : nums) {
            occurrences.put(num, occurrences.getOrDefault(num, 0) + 1);
        }

        List<int[]> values = new ArrayList<int[]>();
        for (Map.Entry<Integer, Integer> entry : occurrences.entrySet()) {
            int num = entry.getKey(), count = entry.getValue();
            values.add(new int[]{num, count});
        }
        int[] ret = new int[k];
        qsort(values, 0, values.size() - 1, ret, 0, k);
        return ret;
    }

    public void qsort(List<int[]> values, int start, int end, int[] ret, int retIndex, int k) {
        int picked = (int) (Math.random() * (end - start + 1)) + start;
        Collections.swap(values, picked, start);
        
        int pivot = values.get(start)[1];
        int index = start;
        for (int i = start + 1; i <= end; i++) {
            if (values.get(i)[1] >= pivot) {
                Collections.swap(values, index + 1, i);
                index++;
            }
        }
        Collections.swap(values, start, index);

        if (k <= index - start) {
            qsort(values, start, index - 1, ret, retIndex, k);
        } else {
            for (int i = start; i <= index; i++) {
                ret[retIndex++] = values.get(i)[0];
            }
            if (k > index - start + 1) {
                qsort(values, index + 1, end, ret, retIndex, k - (index - start + 1));
            }
        }
    }
```

![image-20201205113433099](/Users/lumac/Library/Application Support/typora-user-images/image-20201205113433099.png)

## [统计词频](https://leetcode-cn.com/problems/word-frequency/)



## [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

```java

```



## [根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)



## [分割数组为连续子序列](https://leetcode-cn.com/problems/split-array-into-consecutive-subsequences/)



## [前K个高频单词](https://leetcode-cn.com/problems/top-k-frequent-words/)



## [973. 最接近原点的 K 个点](https://leetcode-cn.com/problems/k-closest-points-to-origin/)



## [最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

```java
 public int[] getLeastNumbers(int[] arr, int k) {
            int low = 0, high = arr.length - 1;
            while (low < high) {
                int i = partition(arr, low, high);
                if (i >= k) high = i - 1;
                if (i <= k) low = i + 1;
            }
            return Arrays.copyOf(arr, k);
        }

        int partition(int[] nums, int low, int high) {
            int pivot = nums[low];
            while (low < high) {
                while (low < high && nums[high] >= pivot)
                    high--;
                nums[low] = nums[high];
                while (low < high && nums[low] <= pivot)
                    low++;
                nums[high] = nums[low];
            }
            nums[low] = pivot;
            return low;
        }
```



## [二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)