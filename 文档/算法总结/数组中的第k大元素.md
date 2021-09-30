#### [215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

```java
 //215. 数组中的第K个最大元素,如果找第K小个元素呢
        public int findKthLargest(int[] nums, int k) {
            return quickSort(nums, 0, nums.length - 1, k - 1);
        }

        //不断的放弃
        public int quickSort(int[] arr, int low, int high, int k) {
            if (low >= high) return arr[low];
            int i = low - 1;//i是向后搜索指针，为简化计算先-1
            int j = high + 1;//j是向前搜索指针，为简化计算下+1
            int pivot = arr[(low + high) / 2];
            while (i < j) {
                do {
                    i++;
                } while (arr[i] > pivot);
                do {
                    j--;
                } while (arr[j] < pivot);
                if (i < j) {
                    int temp = arr[i];
                    arr[i] = arr[j];
                    arr[j] = temp;
                }
            }
            
          ////j的位置:当前索引区间的最大值
            //k:上面k-1了,数组是倒序排的所以k-1就是值,不断的比较就行了
            if (j >= k) { //第k大的元素位于基准左侧
                return quickSort(arr, low, j, k);
            } else { //第k大的元素位于右侧
                return quickSort(arr, j + 1, high, k);
            }
        }
```

