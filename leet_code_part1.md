Let's go through all of them systematically with Java solutions! Here are all the Blind 75 problems for your 3 must-know categories!

---

# 🔴 CATEGORY 1: Arrays & Hashing (9 Problems)

---

### ✅ 1. Two Sum (Easy) — LC #1
```java
public int[] twoSum(int[] nums, int target) {
    HashMap<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** For each number, check if its complement exists in the map
- **Edge case:** What if same element used twice? The map check prevents this ✅

---

### ✅ 2. Contains Duplicate (Easy) — LC #217
```java
public boolean containsDuplicate(int[] nums) {
    HashSet<Integer> set = new HashSet<>();
    for (int num : nums) {
        if (set.contains(num)) return true;
        set.add(num);
    }
    return false;
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** HashSet stores only unique values
- **Edge case:** Empty array returns false ✅

---

### ✅ 3. Valid Anagram (Easy) — LC #242
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (char c : s.toCharArray()) count[c - 'a']++;
    for (char c : t.toCharArray()) count[c - 'a']--;
    for (int i : count) if (i != 0) return false;
    return true;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** Count character frequencies
- **Edge case:** Different lengths → immediately false ✅

---

### ✅ 4. Group Anagrams (Medium) — LC #49
```java
public List<List<String>> groupAnagrams(String[] strs) {
    HashMap<String, List<String>> map = new HashMap<>();
    for (String str : strs) {
        char[] chars = str.toCharArray();
        Arrays.sort(chars);                      // sort each string
        String key = new String(chars);          // sorted string = key
        map.putIfAbsent(key, new ArrayList<>());
        map.get(key).add(str);                   // group by key
    }
    return new ArrayList<>(map.values());
}
```
- **Time:** O(n * k log k) where k = max string length
- **Space:** O(n)
- **Pattern:** Sorted string is the same for all anagrams — use it as a HashMap key
- **Edge case:** Single character strings are their own group ✅

---

### ✅ 5. Top K Frequent Elements (Medium) — LC #347
```java
public int[] topKFrequent(int[] nums, int k) {
    // Step 1 - count frequencies
    HashMap<Integer, Integer> map = new HashMap<>();
    for (int num : nums) {
        map.put(num, map.getOrDefault(num, 0) + 1);
    }

    // Step 2 - bucket sort by frequency
    List<Integer>[] bucket = new List[nums.length + 1];
    for (int key : map.keySet()) {
        int freq = map.get(key);
        if (bucket[freq] == null) bucket[freq] = new ArrayList<>();
        bucket[freq].add(key);
    }

    // Step 3 - collect top k from highest frequency
    int[] result = new int[k];
    int idx = 0;
    for (int i = bucket.length - 1; i >= 0 && idx < k; i--) {
        if (bucket[i] != null) {
            for (int num : bucket[i]) {
                result[idx++] = num;
            }
        }
    }
    return result;
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** Bucket sort — index = frequency, value = list of numbers
- **Edge case:** k always valid, answer is always unique ✅

---

### ✅ 6. Product of Array Except Self (Medium) — LC #238
```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];

    // Step 1 - fill with prefix products (everything to the LEFT)
    result[0] = 1;
    for (int i = 1; i < n; i++) {
        result[i] = result[i - 1] * nums[i - 1];
    }

    // Step 2 - multiply by suffix products (everything to the RIGHT)
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= right;
        right *= nums[i];
    }

    return result;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** Prefix product from left × suffix product from right
- **Edge case:** No division allowed — this solution uses none ✅

---

### ✅ 7. Valid Sudoku (Medium) — LC #36
```java
public boolean isValidSudoku(char[][] board) {
    HashSet<String> seen = new HashSet<>();
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            char num = board[i][j];
            if (num == '.') continue;

            // create unique keys for row, col, and box
            String row = num + " in row " + i;
            String col = num + " in col " + j;
            String box = num + " in box " + i/3 + "-" + j/3;

            // if any key already exists → invalid
            if (!seen.add(row) || !seen.add(col) || !seen.add(box)) {
                return false;
            }
        }
    }
    return true;
}
```
- **Time:** O(1) — board is always 9x9
- **Space:** O(1)
- **Pattern:** Use HashSet with unique string keys for rows, cols, and boxes
- **Edge case:** Empty cells (`.`) are skipped ✅

---

### ✅ 8. Longest Consecutive Sequence (Medium) — LC #128
```java
public int longestConsecutive(int[] nums) {
    HashSet<Integer> set = new HashSet<>();
    for (int num : nums) set.add(num);  // add all to set

    int longest = 0;

    for (int num : set) {
        // only start counting from the BEGINNING of a sequence
        if (!set.contains(num - 1)) {
            int current = num;
            int streak = 1;

            while (set.contains(current + 1)) {
                current++;
                streak++;
            }

            longest = Math.max(longest, streak);
        }
    }
    return longest;
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** Only start counting when you find the start of a sequence
- **Edge case:** Duplicates handled by HashSet ✅

---

### ✅ 9. Encode and Decode Strings (Medium) — LC #271
```java
// Encode
public String encode(List<String> strs) {
    StringBuilder sb = new StringBuilder();
    for (String str : strs) {
        sb.append(str.length()).append('#').append(str);
        // format: "length#word"
        // e.g. "hello" → "5#hello"
    }
    return sb.toString();
}

// Decode
public List<String> decode(String s) {
    List<String> result = new ArrayList<>();
    int i = 0;
    while (i < s.length()) {
        int j = i;
        while (s.charAt(j) != '#') j++;       // find the delimiter
        int len = Integer.parseInt(s.substring(i, j)); // get length
        result.add(s.substring(j + 1, j + 1 + len));  // extract word
        i = j + 1 + len;                               // move forward
    }
    return result;
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** Encode with `length#word` format to handle any special characters
- **Edge case:** Empty strings and strings containing `#` handled correctly ✅

---
---

# 🔴 CATEGORY 2: Dynamic Programming (11 Problems)

---

### ✅ 1. Climbing Stairs (Easy) — LC #70
```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** Exactly like Fibonacci — you already know this! ✅

---

### ✅ 2. House Robber (Medium) — LC #198
```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = nums[0];
    int prev1 = Math.max(nums[0], nums[1]);
    for (int i = 2; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** At each house — max(skip this house, rob this house + 2 back)
- **Edge case:** Single house → return that house's value ✅

---

### ✅ 3. House Robber II (Medium) — LC #213
```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    // run House Robber I twice:
    // once excluding last house, once excluding first house
    return Math.max(
        robRange(nums, 0, nums.length - 2),
        robRange(nums, 1, nums.length - 1)
    );
}

private int robRange(int[] nums, int start, int end) {
    int prev2 = 0, prev1 = 0;
    for (int i = start; i <= end; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** Houses in a circle — split into two linear problems
- **Edge case:** Single house → return its value ✅

---

### ✅ 4. Longest Palindromic Substring (Medium) — LC #5
```java
public String longestPalindrome(String s) {
    String result = "";
    for (int i = 0; i < s.length(); i++) {
        // odd length palindromes
        String odd = expand(s, i, i);
        if (odd.length() > result.length()) result = odd;

        // even length palindromes
        String even = expand(s, i, i + 1);
        if (even.length() > result.length()) result = even;
    }
    return result;
}

private String expand(String s, int left, int right) {
    while (left >= 0 && right < s.length() 
           && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return s.substring(left + 1, right);
}
```
- **Time:** O(n²) **Space:** O(1)
- **Pattern:** Expand around center for both odd and even length palindromes
- **Edge case:** Single character is always a palindrome ✅

---

### ✅ 5. Coin Change (Medium) — LC #322
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // fill with impossible value
    dp[0] = 0;                    // 0 coins needed for amount 0

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```
- **Time:** O(amount × coins) **Space:** O(amount)
- **Pattern:** For each amount, try every coin and take the minimum
- **Edge case:** If amount unreachable → return -1 ✅

---

### ✅ 6. Longest Increasing Subsequence (Medium) — LC #300
```java
public int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);  // every element is a subsequence of length 1
    int max = 1;

    for (int i = 1; i < nums.length; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        max = Math.max(max, dp[i]);
    }
    return max;
}
```
- **Time:** O(n²) **Space:** O(n)
- **Pattern:** For each element, look back at all smaller elements
- **Edge case:** Single element → return 1 ✅

---

### ✅ 7. Word Break (Medium) — LC #139
```java
public boolean wordBreak(String s, List<String> wordDict) {
    HashSet<String> set = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true;  // empty string is always valid

    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && set.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[s.length()];
}
```
- **Time:** O(n²) **Space:** O(n)
- **Pattern:** dp[i] = true if any valid split exists up to index i
- **Edge case:** Empty string returns true ✅

---

### ✅ 8. Combination Sum IV (Medium) — LC #377
```java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;  // one way to make 0 — use nothing

    for (int i = 1; i <= target; i++) {
        for (int num : nums) {
            if (num <= i) {
                dp[i] += dp[i - num];
            }
        }
    }
    return dp[target];
}
```
- **Time:** O(target × n) **Space:** O(target)
- **Pattern:** Count number of ways — similar to Coin Change but adds instead of min
- **Edge case:** dp[0] = 1 is the base case ✅

---

### ✅ 9. Maximum Product Subarray (Medium) — LC #152
```java
public int maxProduct(int[] nums) {
    int maxProd = nums[0];
    int minProd = nums[0];
    int result = nums[0];

    for (int i = 1; i < nums.length; i++) {
        // negative × negative = positive, so track both min and max
        int temp = maxProd;
        maxProd = Math.max(nums[i], Math.max(maxProd * nums[i], minProd * nums[i]));
        minProd = Math.min(nums[i], Math.min(temp * nums[i], minProd * nums[i]));
        result = Math.max(result, maxProd);
    }
    return result;
}
```
- **Time:** O(n) **Space:** O(1)
- **Pattern:** Track both max AND min (negative × negative = positive!)
- **Edge case:** Single element, array with zeros ✅

---

### ✅ 10. Unique Paths (Medium) — LC #62
```java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];

    // first row and column are all 1s (only one way to reach them)
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1]; // from above + from left
        }
    }
    return dp[m-1][n-1];
}
```
- **Time:** O(m×n) **Space:** O(m×n)
- **Pattern:** Each cell = paths from above + paths from left
- **Edge case:** 1×1 grid returns 1 ✅

---

### ✅ 11. Decode Ways (Medium) — LC #91
```java
public int numDecodings(String s) {
    if (s.charAt(0) == '0') return 0;
    int n = s.length();
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        int oneDigit = Integer.parseInt(s.substring(i-1, i));
        int twoDigit = Integer.parseInt(s.substring(i-2, i));

        if (oneDigit >= 1) dp[i] += dp[i-1];        // valid single digit
        if (twoDigit >= 10 && twoDigit <= 26) dp[i] += dp[i-2]; // valid two digits
    }
    return dp[n];
}
```
- **Time:** O(n) **Space:** O(n)
- **Pattern:** At each position check if 1 or 2 digit decode is valid
- **Edge case:** Leading zeros → return 0 ✅

---
---

# 🔴 CATEGORY 3: Binary Search (2 Problems)

---

### ✅ 1. Find Minimum in Rotated Sorted Array (Medium) — LC #153
```java
public int findMin(int[] nums) {
    int left = 0, right = nums.length - 1;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] > nums[right]) {
            left = mid + 1;  // min is in RIGHT half
        } else {
            right = mid;     // min is in LEFT half (including mid)
        }
    }
    return nums[left];
}
```
- **Time:** O(log n) **Space:** O(1)
- **Pattern:** Compare mid to right — if mid > right, rotation is on the right side
- **Edge case:** Already sorted array → returns first element ✅

---

### ✅ 2. Search in Rotated Sorted Array (Medium) — LC #33
```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) return mid;

        // left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;  // target in left half
            } else {
                left = mid + 1;   // target in right half
            }
        }
        // right half is sorted
        else {
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;   // target in right half
            } else {
                right = mid - 1;  // target in left half
            }
        }
    }
    return -1;
}
```
- **Time:** O(log n) **Space:** O(1)
- **Pattern:** One half is always sorted — figure out which half and search there
- **Edge case:** Target not found → return -1 ✅

---

## 📊 Master Tracker — All 22 Problems:

| # | Problem | Category | Difficulty | Tried? | Solved? |
|---|---|---|---|---|---|
| 1 | Two Sum | Arrays | Easy | ⬜ | ⬜ |
| 2 | Contains Duplicate | Arrays | Easy | ⬜ | ⬜ |
| 3 | Valid Anagram | Arrays | Easy | ⬜ | ⬜ |
| 4 | Group Anagrams | Arrays | Medium | ⬜ | ⬜ |
| 5 | Top K Frequent | Arrays | Medium | ⬜ | ⬜ |
| 6 | Product Except Self | Arrays | Medium | ⬜ | ⬜ |
| 7 | Valid Sudoku | Arrays | Medium | ⬜ | ⬜ |
| 8 | Longest Consecutive | Arrays | Medium | ⬜ | ⬜ |
| 9 | Encode & Decode | Arrays | Medium | ⬜ | ⬜ |
| 10 | Climbing Stairs | DP | Easy | ⬜ | ⬜ |
| 11 | House Robber | DP | Medium | ⬜ | ⬜ |
| 12 | House Robber II | DP | Medium | ⬜ | ⬜ |
| 13 | Longest Palindrome | DP | Medium | ⬜ | ⬜ |
| 14 | Coin Change | DP | Medium | ⬜ | ⬜ |
| 15 | Longest Inc. Subseq | DP | Medium | ⬜ | ⬜ |
| 16 | Word Break | DP | Medium | ⬜ | ⬜ |
| 17 | Combination Sum IV | DP | Medium | ⬜ | ⬜ |
| 18 | Max Product Subarray | DP | Medium | ⬜ | ⬜ |
| 19 | Unique Paths | DP | Medium | ⬜ | ⬜ |
| 20 | Decode Ways | DP | Medium | ⬜ | ⬜ |
| 21 | Find Min Rotated | Binary Search | Medium | ⬜ | ⬜ |
| 22 | Search Rotated | Binary Search | Medium | ⬜ | ⬜ |

---

## 💡 How to Study Each Problem:
```
Step 1 → Read the problem carefully
Step 2 → Write out a small example by hand
Step 3 → Try to code it in IntelliJ WITHOUT looking
Step 4 → If stuck after 20 mins → look at the hint only
Step 5 → If still stuck → look at solution and understand WHY
Step 6 → Close the solution and recode it from scratch
Step 7 → Paste it here and I'll review it! ✅
```

Which problem do you want to tackle first? 💪