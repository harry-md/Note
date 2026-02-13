---
title: Top K Frequent Elements
tag: [DSA, Medium, Array, Hash Table, Divide and Conquer, Sorting, Heap (Priority Queue), Bucket Sort, Counting, Quickselect]
---

# 1. Đề bài

Given an integer array nums and an integer k, return the k most frequent elements within the array.

The test cases are generated such that the answer is always unique.

You may return the output in any order.

Example 1:


```
Input: nums = [1,2,2,3,3,3], k = 2

Output: [2,3]
```
Example 2:

```
Input: nums = [7,7], k = 1

Output: [7]
```
Constraints:

```
1 <= nums.length <= 10^4.
-1000 <= nums[i] <= 1000
1 <= k <= number of distinct elements in nums.
```

# 2. Hướng giải

## 2.1 Dùng Max Heap

1. Tạo 1 map với <phần tử, số lần xuất hiện>.
2. Tạo 1 max heap (số lần xuất hiện nhiều nhất đứng đầu, ít nhất đứng cuối).
3. Lấy k phần tử từ max heap.

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i : nums) {
        // phần tử chưa xuất hiện thì đưa số 1 vào, xuất hiện rồi thì +1
        map.merge(i, 1, Integer::sum); 
    }
    
    // map.get(b) - map.get(a) để tạo max-heap sắp xếp dựa trên tần xuất (value của map)
    Queue<Integer> queue = new PriorityQueue<>((a, b) -> map.get(b) - map.get(a));

    // thêm tất cả key vào queue
    queue.addAll(map.keySet());

    int[] res = new int[k];
    for (int i = 0; i < k; i++) {
        // lấy ra k thằng đầu (có tần xuất cao nhất)
        res[i] = queue.poll();
    }
    return res;
}
```

>Time Complexity: O(nlogn) (do add toàn bộ key vào heap), Space Complexity: O(n)

## 2.2 Hướng Bucket Sort

1. Vẫn tạo 1 map đếm tần suất như trên.
2. Tạo bucket. bucket là mảng có index là tần số còn value là số có tần số đó.
3. Duyệt từ cuối bucket, lấy k index (tần số) lớn nhất trở xuống.

```java
public static int[] topKFrequent2(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i : nums) {
        map.merge(i, 1, Integer::sum);
    }

    // bucket có kích thước nums.length+1
    // do tần số cao nhất có thể: [1, 1, 1] -> freq: 3, tạo mảng tới index 3
    List<Integer>[] bucket = new List[nums.length + 1];
    for (int key : map.keySet()) {
        // duyệt qua key của map. thêm vào bucket ở index ứng với tần số của key đó.
        int freq = map.get(key);
        if (bucket[freq] == null) bucket[freq] = new ArrayList<>();
        bucket[freq].add(key);
    }

    int[] res = new int[k];
    int count = 0;
    // duyệt ngược mảng bucket (do index càng lớn - freq càng lớn)
    for (int i = bucket.length - 1; i >= 0 && count < k; i--) {
        if (bucket[i] != null) {
            // do 1 vị trí index có thể chứ nhiều số cùng tần số
            // e.g., [1, 1, 2, 3, 3] -> index 2 chứa [1, 3]
            // duyệt ở index 2 lấy cả 2 số 1 và 3
            for (int j : bucket[i]) {
                res[count++] = j;
                if (count == k) break;
            }
        }
    }

    return res;
}
```

>Time Complexity: O(n)
