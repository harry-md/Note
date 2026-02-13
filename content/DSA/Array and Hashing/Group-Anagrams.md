---
title: Group Anagrams 
---

Độ khó: Medium

# 1. Đề bài

Given an array of strings strs, group all anagrams together into sublists. You may return the output in any order.

An anagram is a string that contains the exact same characters as another string, but the order of the characters can be different.

**Example 1:**

```
Input: strs = ["act","pots","tops","cat","stop","hat"]

Output: [["hat"],["act", "cat"],["stop", "pots", "tops"]]
```

**Example 2:**

```
Input: strs = ["x"]

Output: [["x"]]
```


**Example 3:**

```
Input: strs = [""]

Output: [[""]]
```

# 2. Hướng giải

Bài này có 2 cách giải: Sort Key và Tạo Key Unique

## 2.1 Sort Key

1. Duyệt qua các phần tử, lấy từng phần tử đi sort.

2. Lấy biến đã sort đó cho vào làm key của map. nếu key đã có rồi thì thêm vào list, chưa có thì tạo list và thêm phần từ vào.

3. Lặp lại cho đến hết map.


```java
public List<List<String>> groupAnagrams(String[] strs) {
    if (strs == null || strs.length == 0) return new ArrayList<>();

    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        // sort key
        char[] ca = s.toCharArray();
        Arrays.sort(ca);

        // Key đã được sort
        String key = String.valueOf(ca);
        
        // nếu key chưa có thì tạo list và thêm phần tử vào
        // nếu key đã có thì thêm phần tử vào cái list đó
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }

    return new ArrayList<>(map.values());
}
```

> Time Complexity: O(n * klogk), Space Complexity: O(n)

>NOTE: `computeIfAbsent()` trả về value ở vị trí key.

## 2.2 Tạo Key Unique

1. Tạo 1 mảng tần số các chữ cái trong chuỗi s. Ví dụ `[1, 0, 0, 0, 1]` (`ae`, `ea` đều sẽ ra 2 mảng giống nhau do tần số xuất hiện của từng chữ cái là như nhau - a: 1 lần, e: 1 lần).

2. Tạo 1 chuỗi dạng `1#0#1#...` từ mảng trên. ví dụ chuỗi `ade` -> `1#0#0#1#1` (`tần số chữ a#tần số chữ b#tần số chữ c#...`).

3. Duyệt qua từng phần tử, tạo key từ mỗi phần tử.

4. Nếu key đã có trong map rồi thì thêm phần tử vào list, chưa có thì tạo list và thêm phần từ vào list.

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();

    for (String s : strs) {
        String key = getSignature(s);
        map.computeIfAbsent(key, v -> new ArrayList<>()).add(s);
    }

    return new ArrayList<>(map.values());
}

private static String getSignature(String s) {
    if (s==null && s.isEmpty()) return "";
    int[] map = new int[26];
    for (int i = 0; i < s.length(); i++) {
        int idx = s.charAt(i) - 'a';
        if (map[idx] == 0) {
            map[idx] = 1;
        } else {
            map[idx] += 1;
        }
    }

    // dùng StringBuilder tối ưu tốc độ nối chuỗi
    StringBuilder strBuilder = new StringBuilder();
    for (int i = 0; i < map.length; i++) {
        strBuilder.append(map[i] + "#");
    }
    return strBuilder.toString();
}
```

> Time Complexity: O(n * k), Space Complexity: O(n)
