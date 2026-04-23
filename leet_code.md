Great choices! Let's build a solid foundation. Since you're a beginner, I'll walk you through each topic clearly with explanations, patterns, and Java code examples. Let's start with the one explicitly called out in your interview doc.

---

## 🔺 1. Pascal's Triangle

**What it is:** A triangle where each number is the sum of the two numbers directly above it.

```
    1
   1 1
  1 2 1
 1 3 3 1
1 4 6 4 1
```

**The pattern:** `triangle[row][col] = triangle[row-1][col-1] + triangle[row-1][col]`

### Common LeetCode Problems:

**Problem 1 – Generate Pascal's Triangle (LC #118)**
```java
public List<List<Integer>> generate(int numRows) {
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < numRows; i++) {
        List<Integer> row = new ArrayList<>();
        row.add(1); // first element is always 1

        for (int j = 1; j < i; j++) {
            // sum of two elements above it
            row.add(result.get(i - 1).get(j - 1) + result.get(i - 1).get(j));
        }

        if (i > 0) row.add(1); // last element is always 1
        result.add(row);
    }

    return result;
}
```

**Problem 2 – Return just Row N (LC #119)**
```java
public List<Integer> getRow(int rowIndex) {
    List<Integer> row = new ArrayList<>();
    row.add(1);

    for (int i = 1; i <= rowIndex; i++) {
        // traverse right to left to avoid overwriting values
        for (int j = i - 1; j >= 1; j--) {
            row.set(j, row.get(j - 1) + row.get(j));
        }
        row.add(1);
    }

    return row;
}
```

---

## 📋 2. Arrays & Strings

**Key patterns to know:**

### Two Pointers
Used for problems like reversing, finding pairs, checking palindromes.
```java
// Reverse an array
public void reverse(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

### Sliding Window
Used for subarray/substring problems.
```java
// Max sum of subarray of size k
public int maxSum(int[] arr, int k) {
    int windowSum = 0, maxSum = 0;

    for (int i = 0; i < k; i++) windowSum += arr[i]; // first window
    maxSum = windowSum;

    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k]; // slide window
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}
```

### HashMap for frequency counting
```java
// Check if two strings are anagrams
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    HashMap<Character, Integer> map = new HashMap<>();

    for (char c : s.toCharArray()) map.put(c, map.getOrDefault(c, 0) + 1);
    for (char c : t.toCharArray()) {
        map.put(c, map.getOrDefault(c, 0) - 1);
        if (map.get(c) < 0) return false;
    }

    return true;
}
```

---

## 🧠 3. Dynamic Programming (DP)

**What it is:** Breaking a big problem into smaller overlapping subproblems and storing results to avoid recomputation.

**The mindset:** Ask yourself — *"Does this problem have repeated subproblems?"*

### Classic Example – Fibonacci
```java
// Without DP (slow - recalculates same values)
// With DP (memoization)
public int fib(int n) {
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

### Climbing Stairs (LC #70) — very common beginner DP
```java
// How many ways to climb n stairs, taking 1 or 2 steps at a time?
public int climbStairs(int n) {
    if (n <= 2) return n;
    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2]; // same pattern as Fibonacci!
    }
    return dp[n];
}
```

---

## 🔃 4. Sorting & Searching

### Know these sorts conceptually:
| Algorithm | Time Complexity | Notes |
|---|---|---|
| Bubble Sort | O(n²) | Simple but slow |
| Merge Sort | O(n log n) | Divide & conquer |
| Quick Sort | O(n log n) avg | In-place, fast in practice |
| Binary Search | O(log n) | Only on **sorted** arrays |

### Binary Search (very commonly tested)
```java
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2; // avoids overflow

        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }

    return -1; // not found
}
```

---

## 💡 General Interview Tips

1. **Talk out loud** — explain your thought process before coding
2. **Clarify first** — ask about edge cases (empty input, negatives, duplicates)
3. **Start with brute force** — then optimize
4. **Know Big-O** — be ready to say "this is O(n) time and O(1) space"
5. **Test your code** — walk through a small example manually after writing

---

Would you like me to give you **practice problems** to try on your own, go deeper on any of these topics, or do a **mock interview walkthrough** on one of them?