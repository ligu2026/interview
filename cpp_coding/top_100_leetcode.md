Preparing for coding interviews can be truly overwhelming, especially with the vast number of problems on LeetCode. To help you focus, I've curated the **Top 100 coding interview questions**, organized by category, with their C++ solutions, complexity analysis, and detailed explanations. The selection is based on LeetCode's **"Top 100 Liked Questions"** and the most frequently asked problems by companies like Amazon, Google, and Facebook.

---

### ✅ How to Use This Guide

Reading this guide is only the first step. To truly retain what you learn and build the muscle memory needed for a real interview, I strongly recommend the following active recall strategy:
1.  **Hands-On First**: Read the problem statement, cover the solution, and try to code it yourself.
2.  **Compare & Learn**: Only then, compare your approach with the provided solution.
3.  **Explain Aloud**: Practice explaining your code's logic and complexity out loud, as you would during an interview.

---

### 📋 Table of Contents

To assist your study plan, here is a thematic guide to the most important problem types:

*   [🛠️ 1. Hash Table (5 problems)](#-1-hash-table)
*   [🔄 2. Two Pointers (6 problems)](#-2-two-pointers)
*   [🪟 3. Sliding Window (5 problems)](#-3-sliding-window)
*   [📚 4. Arrays & Strings (11 problems)](#-4-arrays--strings)
*   [📊 5. Matrix (4 problems)](#-5-matrix)
*   [🔗 6. Linked List (12 problems)](#-6-linked-list)
*   [🌳 7. Tree / Binary Search Tree (15 problems)](#-7-tree--binary-search-tree)
*   [🌐 8. Graph Theory (3 problems)](#-8-graph-theory)
*   [🔄 9. Backtracking (8 problems)](#-9-backtracking)
*   [🔎 10. Binary Search (6 problems)](#-10-binary-search)
*   [📚 11. Stack & Queue (6 problems)](#-11-stack--queue)
*   [⛰️ 12. Heap / Priority Queue (5 problems)](#-12-heap--priority-queue)
*   [👤 13. Greedy (5 problems)](#-13-greedy)
*   [🧠 14. Dynamic Programming (9 problems)](#-14-dynamic-programming)

Each subsection contains the key techniques to master and the essential practice problems. For a more detailed breakdown and complete problem list, you can explore external lists like the **Blind 75** sheet.

---

### 🛠️ 1. Hash Table

**Key Techniques**: Master the concept of "space-time tradeoff" by using hash maps for O(1) lookups. Use them to store complements (Two Sum), frequency counts (Majority Element), and for data deduplication.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 1 | **[Two Sum](https://leetcode.com/problems/two-sum/)** | Use a hash map to store each element's index as we iterate. For each `nums[i]`, check if `target - nums[i]` exists in the map. If yes, return the indices. | [C++ Solution](#two-sum) | Easy |
| 49 | **[Group Anagrams](https://leetcode.com/problems/group-anagrams/)** | Sort each string to create a "signature" key, then use a hash map to group strings with the same key. | [C++ Solution](#group-anagrams) | Medium |
| 128 | **[Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)** | Insert all numbers into a hash set. For each number `n`, if `n-1` is not in the set (meaning it's the start of a sequence), count how many consecutive numbers exist from `n` upwards. | [C++ Solution](#longest-consecutive-sequence) | Medium |
| 169 | **[Majority Element](https://leetcode.com/problems/majority-element/)** | Use a hash map to count frequencies. Return the element whose frequency > `n/2`. (Can also be solved with Boyer-Moore Voting Algorithm). | [C++ Solution](#majority-element) | Easy |
| 242 | **[Valid Anagram](https://leetcode.com/problems/valid-anagram/)** | Use a hash map to count characters of `s`, then decrement for `t`. All counts must be zero at the end. | [C++ Solution](#valid-anagram) | Easy |

#### C++ Code Examples


<details>
<summary> 1. Two Sum </summary>

```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> numMap;  // value -> index
        for (int i = 0; i < nums.size(); ++i) {
            int complement = target - nums[i];
            if (numMap.find(complement) != numMap.end()) {
                return {numMap[complement], i};
            }
            numMap[nums[i]] = i;
        }
        return {};  // Should never reach here for valid input
    }
};

// TIME: O(n) - one pass through array
// SPACE: O(n) - hash map stores up to n elements
```
</details>

<details>
<summary> 49. Group Anagrams </summary>

```cpp
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> map;
        for (string& s : strs) {
            string key = s;
            sort(key.begin(), key.end());
            map[key].push_back(s);
        }
        vector<vector<string>> result;
        for (auto& p : map) {
            result.push_back(p.second);
        }
        return result;
    }
};

// TIME: O(n * k log k) where n is number of strings, k is max length
// SPACE: O(n * k) - store all strings in map
```
</details>

<details>
<summary> 128. Longest Consecutive Sequence </summary>

```cpp
#include <vector>
#include <unordered_set>
using namespace std;

class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> numSet(nums.begin(), nums.end());
        int longest = 0;
        for (int n : numSet) {
            // Only start counting if n is the start of a sequence
            if (numSet.find(n - 1) == numSet.end()) {
                int length = 1;
                while (numSet.find(n + length) != numSet.end()) {
                    ++length;
                }
                longest = max(longest, length);
            }
        }
        return longest;
    }
};

// TIME: O(n) - each element visited at most twice
// SPACE: O(n) - hash set stores all elements
```
</details>

<details>
<summary> 169. Majority Element </summary>

```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    int majorityElement(vector<int>& nums) {
        unordered_map<int, int> freq;
        int majority = nums.size() / 2;
        for (int n : nums) {
            if (++freq[n] > majority) return n;
        }
        return -1;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 242. Valid Anagram </summary>

```cpp
#include <string>
#include <array>
using namespace std;

class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.length() != t.length()) return false;
        array<int, 26> count{};
        for (char c : s) count[c - 'a']++;
        for (char c : t) {
            if (--count[c - 'a'] < 0) return false;
        }
        return true;
    }
};

// TIME: O(n)
// SPACE: O(1) - fixed size array (26 charectors)
```
</details>


### 🔄 2. Two Pointers

**Key Techniques**: Solve problems in sorted arrays by moving two pointers from opposite ends (e.g., Container with Most Water) or both from the start (e.g., fast-slow pointers for linked list cycles).

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 11 | **[Container With Most Water](https://leetcode.com/problems/container-with-most-water/)** | Use two pointers `left` and `right` starting at ends. Calculate area = `(right-left) * min(height[left], height[right])`. Move the pointer with the smaller height inward. Track maximum area. | [C++ Solution](#container-with-most-water) | Medium |
| 15 | **[3Sum](https://leetcode.com/problems/3sum/)** | Sort array, iterate with index `i`. For each `i`, use two pointers (`left`, `right`) to find two numbers that sum to `-nums[i]`. Skip duplicates to avoid repeated triplets. | [C++ Solution](#3sum) | Medium |
| 42 | **[Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)** | Use two pointers from both ends and maintain `left_max` and `right_max`. At each step, the pointer with smaller max decides how much water it can trap. | [C++ Solution](#trapping-rain-water) | Hard |
| 125 | **[Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)** | Use two pointers: one from start, one from end. Skip non-alphanumeric characters, compare lowercase versions. | [C++ Solution](#valid-palindrome) | Easy |
| 283 | **[Move Zeroes](https://leetcode.com/problems/move-zeroes/)** | Use a `nonZeroIndex` pointer: iterate through array, when a non-zero is found, swap it to `nonZeroIndex` and increment. | [C++ Solution](#move-zeroes) | Easy |
| 16 | **[3Sum Closest](https://leetcode.com/problems/3sum-closest/)** | Sort array, for each `i` use two pointers to find sum closest to target. Update closest when better sum found. | [C++ Solution](#3sum-closest) | Medium |

#### C++ Code Examples


<details>
<summary> 11. Container With Most Water </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0, right = height.size() - 1;
        int maxWater = 0;
        while (left < right) {
            int currWater = (right - left) * min(height[left], height[right]);
            maxWater = max(maxWater, currWater);
            if (height[left] < height[right]) ++left;
            else --right;
        }
        return maxWater;
    }
};

// TIME: O(n) - single pass
// SPACE: O(1)
```
</details>

<details>
<summary> 15. 3Sum </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        int n = nums.size();
        for (int i = 0; i < n - 2; ++i) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;  // skip duplicates
            int left = i + 1, right = n - 1;
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    result.push_back({nums[i], nums[left], nums[right]});
                    // skip duplicates for left and right
                    while (left < right && nums[left] == nums[left + 1]) ++left;
                    while (left < right && nums[right] == nums[right - 1]) --right;
                    ++left; --right;
                } else if (sum < 0) ++left;
                else --right;
            }
        }
        return result;
    }
};

// TIME: O(n²)
// SPACE: O(1) excluding output
```
</details>

<details>
<summary> 42. Trapping Rain Water </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int trap(vector<int>& height) {
        if (height.empty()) return 0;
        int left = 0, right = height.size() - 1;
        int leftMax = 0, rightMax = 0;
        int water = 0;
        while (left < right) {
            if (height[left] < height[right]) {
                leftMax = max(leftMax, height[left]);
                water += leftMax - height[left];
                ++left;
            } else {
                rightMax = max(rightMax, height[right]);
                water += rightMax - height[right];
                --right;
            }
        }
        return water;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 125. Valid Palindrome </summary>

```cpp
#include <string>
#include <cctype>
using namespace std;

class Solution {
public:
    bool isPalindrome(string s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            while (left < right && !isalnum(s[left])) ++left;
            while (left < right && !isalnum(s[right])) --right;
            if (tolower(s[left]) != tolower(s[right])) return false;
            ++left; --right;
        }
        return true;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 283. Move Zeroes </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int nonZeroIndex = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != 0) {
                swap(nums[nonZeroIndex++], nums[i]);
            }
        }
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 16. 3Sum Closest </summary>

```cpp
#include <vector>
#include <algorithm>
#include <cmath>
using namespace std;

class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        int closest = nums[0] + nums[1] + nums[2];
        int n = nums.size();
        for (int i = 0; i < n - 2; ++i) {
            int left = i + 1, right = n - 1;
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == target) return sum;
                if (abs(sum - target) < abs(closest - target)) closest = sum;
                if (sum < target) ++left;
                else --right;
            }
        }
        return closest;
    }
};

// TIME: O(n²)
// SPACE: O(1)
```
</details>


### 🪟 3. Sliding Window

**Key Techniques**: Ideal for subarray/substring problems. Maintain a window using two pointers (`left`, `right`). Expand `right` to include new elements, contract `left` when the window becomes invalid. Use hash maps to track character frequencies.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 3 | **[Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)** | Slide `right` pointer, use hash set to track characters in current window. When duplicate found, move `left` until duplicate removed. | [C++ Solution](#longest-substring-without-repeating) | Medium |
| 438 | **[Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)** | Use fixed-size sliding window with character count arrays. When window size equals `p.length()`, if counts match → store start index. | [C++ Solution](#find-all-anagrams) | Medium |
| 76 | **[Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)** | Expand `right` to include characters until window contains all needed chars. Then contract `left` to minimal valid window. | [C++ Solution](#minimum-window-substring) | Hard |
| 560 | **[Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)** | Use cumulative sum with a hash map (not exactly sliding window for negatives, but good to know). | [C++ Solution](#subarray-sum-equals-k) | Medium |
| 239 | **[Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)** | Use a deque to maintain indices of potential maximums in current window. Deque stores decreasing values. | [C++ Solution](#sliding-window-maximum) | Hard |

#### C++ Code Examples


<details>
<summary> 3. Longest Substring Without Repeating Characters </summary>

```cpp
#include <string>
#include <unordered_set>
using namespace std;

class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char> window;
        int left = 0, maxLen = 0;
        for (int right = 0; right < s.length(); ++right) {
            while (window.find(s[right]) != window.end()) {
                window.erase(s[left++]);
            }
            window.insert(s[right]);
            maxLen = max(maxLen, right - left + 1);
        }
        return maxLen;
    }
};

// TIME: O(n) - each char added/removed once
// SPACE: O(min(n, charset)) - charset size (e.g., 256 for ASCII)
```
</details>

<details>
<summary> 438. Find All Anagrams in a String </summary>

```cpp
#include <vector>
#include <string>
#include <array>
using namespace std;

class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        if (s.length() < p.length()) return {};
        array<int, 26> pCount{}, sCount{};
        for (char c : p) pCount[c - 'a']++;
        vector<int> result;
        for (int i = 0; i < s.length(); ++i) {
            sCount[s[i] - 'a']++;
            if (i >= p.length()) sCount[s[i - p.length()] - 'a']--;
            if (i >= p.length() - 1 && pCount == sCount) result.push_back(i - p.length() + 1);
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(1) - fixed size arrays
```
</details>

<details>
<summary> 76. Minimum Window Substring </summary>

```cpp
#include <string>
#include <unordered_map>
#include <climits>
using namespace std;

class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char, int> target;
        for (char c : t) target[c]++;
        int required = target.size();
        int left = 0, right = 0, formed = 0;
        unordered_map<char, int> window;
        int minLen = INT_MAX, minLeft = 0;
        while (right < s.length()) {
            char c = s[right];
            window[c]++;
            if (target.find(c) != target.end() && window[c] == target[c]) formed++;
            while (left <= right && formed == required) {
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    minLeft = left;
                }
                char leftChar = s[left];
                window[leftChar]--;
                if (target.find(leftChar) != target.end() && window[leftChar] < target[leftChar]) formed--;
                left++;
            }
            right++;
        }
        return minLen == INT_MAX ? "" : s.substr(minLeft, minLen);
    }
};

// TIME: O(s + t)
// SPACE: O(s + t)
```
</details>

<details>
<summary> 560. Subarray Sum Equals K </summary>

```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> prefixSumCount;
        prefixSumCount[0] = 1;
        int sum = 0, count = 0;
        for (int n : nums) {
            sum += n;
            if (prefixSumCount.find(sum - k) != prefixSumCount.end()) {
                count += prefixSumCount[sum - k];
            }
            prefixSumCount[sum]++;
        }
        return count;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 239. Sliding Window Maximum </summary>

```cpp
#include <vector>
#include <deque>
using namespace std;

class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        deque<int> dq;  // stores indices
        vector<int> result;
        for (int i = 0; i < nums.size(); ++i) {
            // remove indices that are out of window
            while (!dq.empty() && dq.front() < i - k + 1) dq.pop_front();
            // maintain decreasing order: remove smaller elements
            while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back();
            dq.push_back(i);
            if (i >= k - 1) result.push_back(nums[dq.front()]);
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(k)
```
</details>


### 📚 4. Arrays & Strings

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 53 | **[Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)** | Kadane's Algorithm: track `currentSum = max(n, currentSum + n)`, update `maxSum` accordingly. | [C++ Solution](#maximum-subarray) | Easy |
| 56 | **[Merge Intervals](https://leetcode.com/problems/merge-intervals/)** | Sort by start time. Merge overlapping intervals by comparing current start with previous end. | [C++ Solution](#merge-intervals) | Medium |
| 287 | **[Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)** | Use Floyd's Cycle Detection (treat array as linked list where value = next index). | [C++ Solution](#find-duplicate-number) | Medium |
| 238 | **[Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)** | Use two passes: left products then right products, multiply them. (No division). | [C++ Solution](#product-of-array-except-self) | Medium |
| 8 | **[String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)** | Simulate DFA: skip spaces, handle signs, process digits, check overflow. | [C++ Solution](#string-to-integer-atoi) | Medium |
| 14 | **[Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)** | Horizontal scanning: compare string by string, reduce prefix. | [C++ Solution](#longest-common-prefix) | Easy |
| 121 | **[Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)** | Track `minPrice`, compute profit `price - minPrice`, update `maxProfit`. | [C++ Solution](#best-time-to-buy-and-sell-stock) | Easy |
| 41 | **[First Missing Positive](https://leetcode.com/problems/first-missing-positive/)** | Place each positive number `x` at index `x-1` if possible. Then find first index where `nums[i] != i+1`. | [C++ Solution](#first-missing-positive) | Hard |
| 189 | **[Rotate Array](https://leetcode.com/problems/rotate-array/)** | Reverse entire array, then reverse first `k` elements, then reverse rest. | [C++ Solution](#rotate-array) | Medium |
| 217 | **[Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)** | Use hash set to detect duplicates in O(1) time. | [C++ Solution](#contains-duplicate) | Easy |
| 268 | **[Missing Number](https://leetcode.com/problems/missing-number/)** | Use sum formula `n*(n+1)/2` or XOR approach. | [C++ Solution](#missing-number) | Easy |

#### C++ Code Examples


<details>
<summary> 53. Maximum Subarray (Kadane's Algorithm) </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum = nums[0];
        int currentSum = nums[0];
        for (int i = 1; i < nums.size(); ++i) {
            currentSum = max(nums[i], currentSum + nums[i]);
            maxSum = max(maxSum, currentSum);
        }
        return maxSum;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 56. Merge Intervals </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> result;
        for (const auto& interval : intervals) {
            if (result.empty() || result.back()[1] < interval[0]) {
                result.push_back(interval);
            } else {
                result.back()[1] = max(result.back()[1], interval[1]);
            }
        }
        return result;
    }
};

// TIME: O(n log n) due to sorting
// SPACE: O(1) excluding output
```
</details>

<details>
<summary> 287. Find the Duplicate Number </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        // Floyd's Cycle Detection (Tortoise and Hare)
        int slow = nums[0];
        int fast = nums[0];
        // Phase 1: find intersection
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);
        // Phase 2: find cycle entrance
        slow = nums[0];
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    }
};

// TIME: O(n)
// SPACE: O(1) - no extra storage
```
</details>

<details>
<summary> 238. Product of Array Except Self </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, 1);
        // left pass: product of all elements to the left
        int leftProduct = 1;
        for (int i = 0; i < n; ++i) {
            result[i] = leftProduct;
            leftProduct *= nums[i];
        }
        // right pass: multiply by product of all elements to the right
        int rightProduct = 1;
        for (int i = n - 1; i >= 0; --i) {
            result[i] *= rightProduct;
            rightProduct *= nums[i];
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(1) excluding output
```
</details>

<details>
<summary> 8. String to Integer (atoi) </summary>

```cpp
#include <string>
#include <climits>
using namespace std;

class Solution {
public:
    int myAtoi(string s) {
        int i = 0, n = s.length();
        while (i < n && s[i] == ' ') ++i;
        if (i == n) return 0;
        int sign = 1;
        if (s[i] == '+' || s[i] == '-') {
            sign = (s[i] == '-') ? -1 : 1;
            ++i;
        }
        long result = 0;
        while (i < n && isdigit(s[i])) {
            result = result * 10 + (s[i] - '0');
            if (result * sign > INT_MAX) return INT_MAX;
            if (result * sign < INT_MIN) return INT_MIN;
            ++i;
        }
        return result * sign;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 14. Longest Common Prefix </summary>

```cpp
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty()) return "";
        string prefix = strs[0];
        for (int i = 1; i < strs.size(); ++i) {
            while (strs[i].find(prefix) != 0) {
                prefix = prefix.substr(0, prefix.length() - 1);
                if (prefix.empty()) return "";
            }
        }
        return prefix;
    }
};

// TIME: O(n * m) where m is prefix length
// SPACE: O(1)
```
</details>

<details>
<summary> 121. Best Time to Buy and Sell Stock </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int minPrice = INT_MAX;
        int maxProfit = 0;
        for (int price : prices) {
            minPrice = min(minPrice, price);
            maxProfit = max(maxProfit, price - minPrice);
        }
        return maxProfit;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 41. First Missing Positive </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
                swap(nums[i], nums[nums[i] - 1]);
            }
        }
        for (int i = 0; i < n; ++i) {
            if (nums[i] != i + 1) return i + 1;
        }
        return n + 1;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 189. Rotate Array </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 217. Contains Duplicate </summary>

```cpp
#include <vector>
#include <unordered_set>
using namespace std;

class Solution {
public:
    bool containsDuplicate(vector<int>& nums) {
        unordered_set<int> seen;
        for (int n : nums) {
            if (seen.find(n) != seen.end()) return true;
            seen.insert(n);
        }
        return false;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 268. Missing Number </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int n = nums.size();
        int expectedSum = n * (n + 1) / 2;
        int actualSum = 0;
        for (int num : nums) actualSum += num;
        return expectedSum - actualSum;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>


### 📊 5. Matrix

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 54 | **[Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)** | Use four boundaries: `top`, `bottom`, `left`, `right`. Traverse in spiral order while shrinking boundaries. | [C++ Solution](#spiral-matrix) | Medium |
| 48 | **[Rotate Image](https://leetcode.com/problems/rotate-image/)** | Step 1: transpose matrix (swap `matrix[i][j]` with `matrix[j][i]`). Step 2: reverse each row. | [C++ Solution](#rotate-image) | Medium |
| 240 | **[Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)** | Start from top-right corner. If `target < current` → move left; if `target > current` → move down. | [C++ Solution](#search-2d-matrix-ii) | Medium |
| 73 | **[Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)** | Use first row and first column as markers. Track if first row/col originally had zeros separately. Then set zeros accordingly. | [C++ Solution](#set-matrix-zeroes) | Medium |

#### C++ Code Examples


<details>
<summary> 54. Spiral Matrix </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> result;
        if (matrix.empty()) return result;
        int top = 0, bottom = matrix.size() - 1;
        int left = 0, right = matrix[0].size() - 1;
        while (top <= bottom && left <= right) {
            // left -> right on top row
            for (int j = left; j <= right; ++j) result.push_back(matrix[top][j]);
            top++;
            // top -> bottom on right column
            for (int i = top; i <= bottom; ++i) result.push_back(matrix[i][right]);
            right--;
            if (top <= bottom) {
                // right -> left on bottom row
                for (int j = right; j >= left; --j) result.push_back(matrix[bottom][j]);
                bottom--;
            }
            if (left <= right) {
                // bottom -> top on left column
                for (int i = bottom; i >= top; --i) result.push_back(matrix[i][left]);
                left++;
            }
        }
        return result;
    }
};

// TIME: O(m * n)
// SPACE: O(1) excluding output
```
</details>

<details>
<summary> 48. Rotate Image </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // transpose
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }
        // reverse each row
        for (int i = 0; i < n; ++i) {
            reverse(matrix[i].begin(), matrix[i].end());
        }
    }
};

// TIME: O(n²)
// SPACE: O(1)
```
</details>

<details>
<summary> 240. Search a 2D Matrix II </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if (matrix.empty()) return false;
        int row = 0;
        int col = matrix[0].size() - 1;
        while (row < matrix.size() && col >= 0) {
            if (matrix[row][col] == target) return true;
            if (matrix[row][col] < target) ++row;
            else --col;
        }
        return false;
    }
};

// TIME: O(m + n)
// SPACE: O(1)
```
</details>

<details>
<summary> 73. Set Matrix Zeroes </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        bool firstRowZero = false, firstColZero = false;
        // check if first row/col need to be zeroed
        for (int j = 0; j < n; ++j) if (!matrix[0][j]) firstRowZero = true;
        for (int i = 0; i < m; ++i) if (!matrix[i][0]) firstColZero = true;
        // use first row and col as markers
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                if (!matrix[i][j]) matrix[0][j] = matrix[i][0] = 0;
            }
        }
        // set zeroes based on markers
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                if (!matrix[0][j] || !matrix[i][0]) matrix[i][j] = 0;
            }
        }
        // zero first row/col if needed
        if (firstRowZero) for (int j = 0; j < n; ++j) matrix[0][j] = 0;
        if (firstColZero) for (int i = 0; i < m; ++i) matrix[i][0] = 0;
    }
};

// TIME: O(m * n)
// SPACE: O(1)
```
</details>


### 🔗 6. Linked List

**Key Techniques**: For linked list problems, master manipulating `next` pointers, dummy heads (to simplify edge cases), and Floyd's Tortoise and Hare algorithm for cycle detection. Know how to reverse a list in-place, find the middle, and merge two sorted lists.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 206 | **[Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)** | Iterative: track `prev`, `curr`, and `nextTemp`. Point `curr->next` to `prev`, move all pointers forward. | [C++ Solution](#reverse-linked-list) | Easy |
| 21 | **[Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)** | Use a dummy head. Compare `list1->val` and `list2->val`, attach smaller to result, advance that pointer. | [C++ Solution](#merge-two-sorted-lists) | Easy |
| 141 | **[Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)** | Floyd's Tortoise and Hare: `slow` moves 1 step, `fast` moves 2 steps. If they meet → cycle exists. | [C++ Solution](#linked-list-cycle) | Easy |
| 142 | **[Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)** | First detect cycle with Tortoise and Hare. Then reset `slow` to head, move both one step until they meet → cycle start. | [C++ Solution](#linked-list-cycle-ii) | Medium |
| 19 | **[Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)** | Use dummy head. `fast` moves `n` steps ahead, then moves both `fast` and `slow` until `fast` reaches end. `slow->next` is the target. | [C++ Solution](#remove-nth-node) | Easy |
| 2 | **[Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)** | Traverse both lists, sum nodes with carry. Create new node for `sum % 10`, continue with `carry = sum / 10`. | [C++ Solution](#add-two-numbers) | Medium |
| 23 | **[Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)** | Use min-heap (priority queue) to always pick smallest among k heads. | [C++ Solution](#merge-k-sorted-lists) | Hard |
| 146 | **[LRU Cache](https://leetcode.com/problems/lru-cache/)** | Combine hash map (for O(1) get) with doubly linked list (for tracking usage order). | [C++ Solution](#lru-cache) | Medium |
| 160 | **[Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/)** | Two pointers traverse both lists: when one reaches end, redirect to the other's head. They meet at intersection. | [C++ Solution](#intersection-of-two-linked-lists) | Easy |
| 234 | **[Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)** | 1) Find middle (fast-slow), 2) Reverse second half, 3) Compare halves, 4) Restore/reverse back (optional). | [C++ Solution](#palindrome-linked-list) | Easy |
| 138 | **[Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)** | Three-pass approach: 1) interweave copy nodes, 2) set random pointers, 3) separate lists. | [C++ Solution](#copy-list-with-random-pointer) | Medium |
| 148 | **[Sort List](https://leetcode.com/problems/sort-list/)** | Merge Sort on linked list: find middle (fast-slow), split, sort recursively, merge. | [C++ Solution](#sort-list) | Medium |

#### C++ Code Examples


<details>
<summary> 206. Reverse Linked List </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr != nullptr) {
            ListNode* nextTemp = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 21. Merge Two Sorted Lists </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode dummy;
        ListNode* tail = &dummy;
        while (list1 && list2) {
            if (list1->val < list2->val) {
                tail->next = list1;
                list1 = list1->next;
            } else {
                tail->next = list2;
                list2 = list2->next;
            }
            tail = tail->next;
        }
        tail->next = list1 ? list1 : list2;
        return dummy.next;
    }
};

// TIME: O(m + n)
// SPACE: O(1)
```
</details>

<details>
<summary> 141. Linked List Cycle </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    bool hasCycle(ListNode* head) {
        if (!head) return false;
        ListNode* slow = head;
        ListNode* fast = head->next;
        while (fast && fast->next) {
            if (slow == fast) return true;
            slow = slow->next;
            fast = fast->next->next;
        }
        return false;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 142. Linked List Cycle II </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        ListNode* slow = head;
        ListNode* fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast) {
                slow = head;
                while (slow != fast) {
                    slow = slow->next;
                    fast = fast->next;
                }
                return slow;
            }
        }
        return nullptr;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 19. Remove Nth Node From End of List </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode dummy(0);
        dummy.next = head;
        ListNode* fast = &dummy;
        ListNode* slow = &dummy;
        for (int i = 0; i < n; ++i) fast = fast->next;
        while (fast->next) {
            fast = fast->next;
            slow = slow->next;
        }
        slow->next = slow->next->next;
        return dummy.next;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 2. Add Two Numbers </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy;
        ListNode* tail = &dummy;
        int carry = 0;
        while (l1 || l2 || carry) {
            int sum = carry;
            if (l1) { sum += l1->val; l1 = l1->next; }
            if (l2) { sum += l2->val; l2 = l2->next; }
            carry = sum / 10;
            tail->next = new ListNode(sum % 10);
            tail = tail->next;
        }
        return dummy.next;
    }
};

// TIME: O(max(m, n))
// SPACE: O(max(m, n))
```
</details>

<details>
<summary> 23. Merge k Sorted Lists </summary>

```cpp
#include <vector>
#include <queue>
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

struct Compare {
    bool operator()(ListNode* a, ListNode* b) { return a->val > b->val; }
};

class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*, vector<ListNode*>, Compare> pq;
        for (ListNode* list : lists) {
            if (list) pq.push(list);
        }
        ListNode dummy;
        ListNode* tail = &dummy;
        while (!pq.empty()) {
            ListNode* smallest = pq.top();
            pq.pop();
            tail->next = smallest;
            tail = tail->next;
            if (smallest->next) pq.push(smallest->next);
        }
        return dummy.next;
    }
};

// TIME: O(n log k) where n is total nodes, k is number of lists
// SPACE: O(k)
```
</details>

<details>
<summary> 146. LRU Cache </summary>

```cpp
#include <unordered_map>
#include <list>
using namespace std;

class LRUCache {
private:
    int capacity;
    list<pair<int, int>> cacheList;  // (key, value)
    unordered_map<int, list<pair<int, int>>::iterator> cacheMap;

public:
    LRUCache(int capacity) : capacity(capacity) {}
    
    int get(int key) {
        if (cacheMap.find(key) == cacheMap.end()) return -1;
        cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
        return cacheMap[key]->second;
    }
    
    void put(int key, int value) {
        if (cacheMap.find(key) != cacheMap.end()) {
            cacheMap[key]->second = value;
            cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
            return;
        }
        if (cacheList.size() >= capacity) {
            int lruKey = cacheList.back().first;
            cacheMap.erase(lruKey);
            cacheList.pop_back();
        }
        cacheList.emplace_front(key, value);
        cacheMap[key] = cacheList.begin();
    }
};

// TIME: O(1) for both get and put
// SPACE: O(capacity)
```
</details>

<details>
<summary> 160. Intersection of Two Linked Lists </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
        if (!headA || !headB) return nullptr;
        ListNode* a = headA;
        ListNode* b = headB;
        while (a != b) {
            a = a ? a->next : headB;
            b = b ? b->next : headA;
        }
        return a;
    }
};

// TIME: O(m + n)
// SPACE: O(1)
```
</details>

<details>
<summary> 234. Palindrome Linked List </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (!head || !head->next) return true;
        // 1) find middle
        ListNode* slow = head;
        ListNode* fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        // 2) reverse second half
        ListNode* prev = nullptr;
        while (slow) {
            ListNode* nextTemp = slow->next;
            slow->next = prev;
            prev = slow;
            slow = nextTemp;
        }
        // 3) compare
        ListNode* left = head;
        ListNode* right = prev;
        while (right) {
            if (left->val != right->val) return false;
            left = left->next;
            right = right->next;
        }
        return true;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 138. Copy List with Random Pointer </summary>

```cpp
#include <unordered_map>
#include <cstddef>
using namespace std;

class Node {
public:
    int val;
    Node* next;
    Node* random;
    Node(int _val) : val(_val), next(nullptr), random(nullptr) {}
};

class Solution {
public:
    Node* copyRandomList(Node* head) {
        if (!head) return nullptr;
        // interweave copied nodes
        Node* curr = head;
        while (curr) {
            Node* copy = new Node(curr->val);
            copy->next = curr->next;
            curr->next = copy;
            curr = copy->next;
        }
        // set random pointers
        curr = head;
        while (curr) {
            if (curr->random) {
                curr->next->random = curr->random->next;
            }
            curr = curr->next->next;
        }
        // separate lists
        Node* dummy = new Node(0);
        Node* copyCurr = dummy;
        curr = head;
        while (curr) {
            copyCurr->next = curr->next;
            copyCurr = copyCurr->next;
            curr->next = curr->next->next;
            curr = curr->next;
        }
        return dummy->next;
    }
};

// TIME: O(n)
// SPACE: O(1) excluding output
```
</details>

<details>
<summary> 148. Sort List </summary>

```cpp
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head || !head->next) return head;
        // find middle
        ListNode* slow = head;
        ListNode* fast = head->next;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        ListNode* mid = slow->next;
        slow->next = nullptr;
        return merge(sortList(head), sortList(mid));
    }
    
    ListNode* merge(ListNode* l1, ListNode* l2) {
        ListNode dummy;
        ListNode* tail = &dummy;
        while (l1 && l2) {
            if (l1->val < l2->val) {
                tail->next = l1;
                l1 = l1->next;
            } else {
                tail->next = l2;
                l2 = l2->next;
            }
            tail = tail->next;
        }
        tail->next = l1 ? l1 : l2;
        return dummy.next;
    }
};

// TIME: O(n log n)
// SPACE: O(log n) for recursion stack
```
</details>


### 🌳 7. Tree / Binary Search Tree

**Key Techniques**: Understand recursion thoroughly for tree traversals (pre-order, in-order, post-order, level-order). Master DFS (depth-first search) and BFS (breadth-first search). For BSTs, utilize the property that in-order traversal yields sorted order.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 94 | **[Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)** | Recursive: left → node → right. Iterative: use stack. | [C++ Solution](#binary-tree-inorder-traversal) | Easy |
| 104 | **[Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)** | Recursive: `1 + max(maxDepth(left), maxDepth(right))`. | [C++ Solution](#maximum-depth-of-binary-tree) | Easy |
| 226 | **[Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/)** | Post-order traversal: swap left and right children recursively. | [C++ Solution](#invert-binary-tree) | Easy |
| 101 | **[Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)** | Recursively check: left's left == right's right AND left's right == right's left. | [C++ Solution](#symmetric-tree) | Easy |
| 543 | **[Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)** | For each node, diameter = max of leftDepth + rightDepth. Use global variable to track max. | [C++ Solution](#diameter-of-binary-tree) | Easy |
| 102 | **[Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)** | BFS with queue: process level by level. | [C++ Solution](#binary-tree-level-order-traversal) | Medium |
| 108 | **[Convert Sorted Array to BST](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)** | Recursively pick middle element as root, build left from left half, right from right half. | [C++ Solution](#convert-sorted-array-to-bst) | Easy |
| 98 | **[Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)** | In-order traversal should yield strictly increasing values. Or use min/max constraints recursively. | [C++ Solution](#validate-binary-search-tree) | Medium |
| 230 | **[Kth Smallest Element in BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)** | In-order traversal: count nodes visited, return when count == k. | [C++ Solution](#kth-smallest-element-in-bst) | Medium |
| 199 | **[Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)** | BFS: push rightmost node of each level into result. | [C++ Solution](#binary-tree-right-side-view) | Medium |
| 114 | **[Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)** | Recursive: flatten left, flatten right. Attach left to right, then append original right. | [C++ Solution](#flatten-binary-tree-to-linked-list) | Medium |
| 105 | **[Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)** | Root is first element of preorder. Find in inorder to split left and right subtrees. | [C++ Solution](#construct-binary-tree) | Medium |
| 437 | **[Path Sum III](https://leetcode.com/problems/path-sum-iii/)** | Prefix sum with hash map: while traversing, check if currentSum - target exists in seen sums. | [C++ Solution](#path-sum-iii) | Medium |
| 236 | **[Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)** | Recursively search for p and q. If both found in different subtrees → current node is LCA. | [C++ Solution](#lowest-common-ancestor) | Medium |
| 124 | **[Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)** | At each node, consider max path through node: left + node + right. Return max single branch to parent. | [C++ Solution](#binary-tree-maximum-path-sum) | Hard |

#### C++ Code Examples


<details>
<summary> 94. Binary Tree Inorder Traversal </summary>

```cpp
#include <vector>
#include <stack>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode*> stk;
        TreeNode* curr = root;
        while (curr || !stk.empty()) {
            while (curr) {
                stk.push(curr);
                curr = curr->left;
            }
            curr = stk.top();
            stk.pop();
            result.push_back(curr->val);
            curr = curr->right;
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 104. Maximum Depth of Binary Tree </summary>

```cpp
#include <algorithm>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};

// TIME: O(n)
// SPACE: O(n) for recursion stack
```
</details>

<details>
<summary> 226. Invert Binary Tree </summary>

```cpp
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root) return nullptr;
        TreeNode* left = invertTree(root->left);
        TreeNode* right = invertTree(root->right);
        root->left = right;
        root->right = left;
        return root;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 101. Symmetric Tree </summary>

```cpp
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (!root) return true;
        return isMirror(root->left, root->right);
    }
    
    bool isMirror(TreeNode* left, TreeNode* right) {
        if (!left && !right) return true;
        if (!left || !right) return false;
        return (left->val == right->val) &&
               isMirror(left->left, right->right) &&
               isMirror(left->right, right->left);
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 543. Diameter of Binary Tree </summary>

```cpp
#include <algorithm>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int diameter = 0;
        height(root, diameter);
        return diameter;
    }
    
    int height(TreeNode* node, int& diameter) {
        if (!node) return 0;
        int leftHeight = height(node->left, diameter);
        int rightHeight = height(node->right, diameter);
        diameter = max(diameter, leftHeight + rightHeight);
        return 1 + max(leftHeight, rightHeight);
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 102. Binary Tree Level Order Traversal </summary>

```cpp
#include <vector>
#include <queue>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        if (!root) return result;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int levelSize = q.size();
            vector<int> level;
            for (int i = 0; i < levelSize; ++i) {
                TreeNode* node = q.front();
                q.pop();
                level.push_back(node->val);
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            result.push_back(level);
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 108. Convert Sorted Array to BST </summary>

```cpp
#include <vector>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return build(nums, 0, nums.size() - 1);
    }
    
    TreeNode* build(vector<int>& nums, int left, int right) {
        if (left > right) return nullptr;
        int mid = left + (right - left) / 2;
        TreeNode* node = new TreeNode(nums[mid]);
        node->left = build(nums, left, mid - 1);
        node->right = build(nums, mid + 1, right);
        return node;
    }
};

// TIME: O(n)
// SPACE: O(n) for recursion stack
```
</details>

<details>
<summary> 98. Validate Binary Search Tree </summary>

```cpp
#include <cstddef>
#include <climits>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    bool isValidBST(TreeNode* root) {
        return validate(root, LONG_MIN, LONG_MAX);
    }
    
    bool validate(TreeNode* node, long minVal, long maxVal) {
        if (!node) return true;
        if (node->val <= minVal || node->val >= maxVal) return false;
        return validate(node->left, minVal, node->val) &&
               validate(node->right, node->val, maxVal);
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 230. Kth Smallest Element in BST </summary>

```cpp
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        int result = 0;
        int count = 0;
        inorder(root, k, count, result);
        return result;
    }
    
    void inorder(TreeNode* node, int k, int& count, int& result) {
        if (!node) return;
        inorder(node->left, k, count, result);
        if (++count == k) result = node->val;
        inorder(node->right, k, count, result);
    }
};

// TIME: O(n) worst-case, O(k) average
// SPACE: O(n)
```
</details>

<details>
<summary> 199. Binary Tree Right Side View </summary>

```cpp
#include <vector>
#include <queue>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> result;
        if (!root) return result;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int levelSize = q.size();
            for (int i = 0; i < levelSize; ++i) {
                TreeNode* node = q.front();
                q.pop();
                if (i == levelSize - 1) result.push_back(node->val);
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 114. Flatten Binary Tree to Linked List </summary>

```cpp
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    void flatten(TreeNode* root) {
        if (!root) return;
        flatten(root->left);
        flatten(root->right);
        TreeNode* left = root->left;
        TreeNode* right = root->right;
        root->left = nullptr;
        root->right = left;
        TreeNode* curr = root;
        while (curr->right) curr = curr->right;
        curr->right = right;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 105. Construct Binary Tree from Preorder and Inorder Traversal </summary>

```cpp
#include <vector>
#include <unordered_map>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        unordered_map<int, int> inorderMap;
        for (int i = 0; i < inorder.size(); ++i) inorderMap[inorder[i]] = i;
        return build(preorder, 0, preorder.size() - 1, inorder, 0, inorder.size() - 1, inorderMap);
    }
    
    TreeNode* build(vector<int>& preorder, int preStart, int preEnd,
                    vector<int>& inorder, int inStart, int inEnd,
                    unordered_map<int, int>& inorderMap) {
        if (preStart > preEnd || inStart > inEnd) return nullptr;
        TreeNode* root = new TreeNode(preorder[preStart]);
        int rootIndex = inorderMap[root->val];
        int leftSize = rootIndex - inStart;
        root->left = build(preorder, preStart + 1, preStart + leftSize,
                           inorder, inStart, rootIndex - 1, inorderMap);
        root->right = build(preorder, preStart + leftSize + 1, preEnd,
                            inorder, rootIndex + 1, inEnd, inorderMap);
        return root;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 437. Path Sum III </summary>

```cpp
#include <unordered_map>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    int pathSum(TreeNode* root, int targetSum) {
        unordered_map<long, int> prefixSumCount;
        prefixSumCount[0] = 1;
        int count = 0;
        dfs(root, 0, targetSum, prefixSumCount, count);
        return count;
    }
    
    void dfs(TreeNode* node, long currentSum, int targetSum,
             unordered_map<long, int>& prefixSumCount, int& count) {
        if (!node) return;
        currentSum += node->val;
        if (prefixSumCount.find(currentSum - targetSum) != prefixSumCount.end()) {
            count += prefixSumCount[currentSum - targetSum];
        }
        prefixSumCount[currentSum]++;
        dfs(node->left, currentSum, targetSum, prefixSumCount, count);
        dfs(node->right, currentSum, targetSum, prefixSumCount, count);
        prefixSumCount[currentSum]--;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 236. Lowest Common Ancestor of a Binary Tree </summary>

```cpp
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (!root || root == p || root == q) return root;
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);
        if (left && right) return root;
        return left ? left : right;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 124. Binary Tree Maximum Path Sum </summary>

```cpp
#include <algorithm>
#include <climits>
#include <cstddef>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int maxSum = INT_MIN;
        maxGain(root, maxSum);
        return maxSum;
    }
    
    int maxGain(TreeNode* node, int& maxSum) {
        if (!node) return 0;
        int leftGain = max(maxGain(node->left, maxSum), 0);
        int rightGain = max(maxGain(node->right, maxSum), 0);
        int pathSum = node->val + leftGain + rightGain;
        maxSum = max(maxSum, pathSum);
        return node->val + max(leftGain, rightGain);
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>


### 🌐 8. Graph Theory

**Key Techniques**: Master both DFS (recursive/stack) and BFS (queue) for traversal through grids (like Number of Islands) or adjacency lists (like Course Schedule). Understand how to track visited nodes to avoid infinite loops.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 200 | **[Number of Islands](https://leetcode.com/problems/number-of-islands/)** | BFS or DFS on each '1' cell, mark visited by changing '1' to '0'. Count times BFS/DFS is started. | [C++ Solution](#number-of-islands) | Medium |
| 207 | **[Course Schedule](https://leetcode.com/problems/course-schedule/)** | Detect cycle in directed graph using topological sort (Kahn's algorithm with in-degree counts). | [C++ Solution](#course-schedule) | Medium |
| 208 | **[Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/)** | Each node has array of 26 children and `isEnd` flag. `insert`, `search`, `startsWith` operations. | [C++ Solution](#implement-trie) | Medium |

#### C++ Code Examples


<details>
<summary> 200. Number of Islands </summary>

```cpp
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        if (grid.empty()) return 0;
        int rows = grid.size(), cols = grid[0].size();
        int islands = 0;
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                if (grid[i][j] == '1') {
                    islands++;
                    dfs(grid, i, j);
                }
            }
        }
        return islands;
    }
    
    void dfs(vector<vector<char>>& grid, int i, int j) {
        if (i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() || grid[i][j] != '1') return;
        grid[i][j] = '0';
        dfs(grid, i + 1, j);
        dfs(grid, i - 1, j);
        dfs(grid, i, j + 1);
        dfs(grid, i, j - 1);
    }
};

// TIME: O(m * n)
// SPACE: O(m * n) worst-case recursion stack
```
</details>

<details>
<summary> 207. Course Schedule </summary>

```cpp
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adj(numCourses);
        vector<int> inDegree(numCourses, 0);
        for (const auto& prereq : prerequisites) {
            adj[prereq[1]].push_back(prereq[0]);
            inDegree[prereq[0]]++;
        }
        queue<int> q;
        for (int i = 0; i < numCourses; ++i) {
            if (inDegree[i] == 0) q.push(i);
        }
        int coursesTaken = 0;
        while (!q.empty()) {
            int course = q.front();
            q.pop();
            coursesTaken++;
            for (int next : adj[course]) {
                if (--inDegree[next] == 0) q.push(next);
            }
        }
        return coursesTaken == numCourses;
    }
};

// TIME: O(V + E)
// SPACE: O(V + E)
```
</details>

<details>
<summary> 208. Implement Trie (Prefix Tree) </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class TrieNode {
public:
    vector<TrieNode*> children;
    bool isEnd;
    TrieNode() : children(26, nullptr), isEnd(false) {}
};

class Trie {
private:
    TrieNode* root;
public:
    Trie() : root(new TrieNode()) {}
    
    void insert(string word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEnd = true;
    }
    
    bool search(string word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->isEnd;
    }
    
    bool startsWith(string prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return true;
    }
};

// TIME: O(len) for each operation
// SPACE: O(total characters)
```
</details>


### 🔄 9. Backtracking

**Key Techniques**: This is essentially DFS for combinatorial search. The core pattern is: `if (condition)` add to result and return / continue; `for (choices)` choose, recurse, unchoose. Use pruning when possible.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 46 | **[Permutations](https://leetcode.com/problems/permutations/)** | Backtrack: track used elements. For each unused element, add to current, recurse, remove. | [C++ Solution](#permutations) | Medium |
| 78 | **[Subsets](https://leetcode.com/problems/subsets/)** | Backtrack: at each index, either include it or not. Append current subset at each step. | [C++ Solution](#subsets) | Medium |
| 17 | **[Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)** | Backtrack: map digits to letters, recursively build combinations. | [C++ Solution](#letter-combinations) | Medium |
| 39 | **[Combination Sum](https://leetcode.com/problems/combination-sum/)** | Backtrack: at each index, choose to include it (can reuse same element) or move to next index. | [C++ Solution](#combination-sum) | Medium |
| 22 | **[Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)** | Backtrack: track `open` and `close` counts. Add '(' if open < n, add ')' if close < open. | [C++ Solution](#generate-parentheses) | Medium |
| 79 | **[Word Search](https://leetcode.com/problems/word-search/)** | Backtrack on grid: for each cell, if first char matches, DFS in 4 directions. Mark visited temporarily. | [C++ Solution](#word-search) | Medium |
| 131 | **[Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)** | Backtrack: at each step, try substrings from current index. If palindrome, partition, recurse. | [C++ Solution](#palindrome-partitioning) | Medium |
| 51 | **[N-Queens](https://leetcode.com/problems/n-queens/)** | Place queens row by row. Track occupied columns, diagonals (row-col and row+col). | [C++ Solution](#n-queens) | Hard |

#### C++ Code Examples


<details>
<summary> 46. Permutations </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> current;
        vector<bool> used(nums.size(), false);
        backtrack(nums, used, current, result);
        return result;
    }
    
    void backtrack(vector<int>& nums, vector<bool>& used, vector<int>& current,
                   vector<vector<int>>& result) {
        if (current.size() == nums.size()) {
            result.push_back(current);
            return;
        }
        for (int i = 0; i < nums.size(); ++i) {
            if (used[i]) continue;
            used[i] = true;
            current.push_back(nums[i]);
            backtrack(nums, used, current, result);
            current.pop_back();
            used[i] = false;
        }
    }
};

// TIME: O(n! * n)
// SPACE: O(n)
```
</details>

<details>
<summary> 78. Subsets </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> result;
        vector<int> current;
        backtrack(nums, 0, current, result);
        return result;
    }
    
    void backtrack(vector<int>& nums, int start, vector<int>& current,
                   vector<vector<int>>& result) {
        result.push_back(current);
        for (int i = start; i < nums.size(); ++i) {
            current.push_back(nums[i]);
            backtrack(nums, i + 1, current, result);
            current.pop_back();
        }
    }
};

// TIME: O(2^n * n)
// SPACE: O(n)
```
</details>

<details>
<summary> 17. Letter Combinations of a Phone Number </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<string> letterCombinations(string digits) {
        if (digits.empty()) return {};
        vector<string> mapping = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
        vector<string> result;
        string current;
        backtrack(digits, 0, mapping, current, result);
        return result;
    }
    
    void backtrack(string& digits, int index, vector<string>& mapping,
                   string& current, vector<string>& result) {
        if (index == digits.length()) {
            result.push_back(current);
            return;
        }
        string& letters = mapping[digits[index] - '0'];
        for (char c : letters) {
            current.push_back(c);
            backtrack(digits, index + 1, mapping, current, result);
            current.pop_back();
        }
    }
};

// TIME: O(4^n * n)
// SPACE: O(n)
```
</details>

<details>
<summary> 39. Combination Sum </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> result;
        vector<int> current;
        backtrack(candidates, target, 0, 0, current, result);
        return result;
    }
    
    void backtrack(vector<int>& candidates, int target, int start, int sum,
                   vector<int>& current, vector<vector<int>>& result) {
        if (sum == target) {
            result.push_back(current);
            return;
        }
        if (sum > target) return;
        for (int i = start; i < candidates.size(); ++i) {
            current.push_back(candidates[i]);
            backtrack(candidates, target, i, sum + candidates[i], current, result);
            current.pop_back();
        }
    }
};

// TIME: O(2^n)
// SPACE: O(n)
```
</details>

<details>
<summary> 22. Generate Parentheses </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        string current;
        backtrack(n, 0, 0, current, result);
        return result;
    }
    
    void backtrack(int n, int open, int close, string& current, vector<string>& result) {
        if (current.length() == 2 * n) {
            result.push_back(current);
            return;
        }
        if (open < n) {
            current.push_back('(');
            backtrack(n, open + 1, close, current, result);
            current.pop_back();
        }
        if (close < open) {
            current.push_back(')');
            backtrack(n, open, close + 1, current, result);
            current.pop_back();
        }
    }
};

// TIME: O(4^n / √n)
// SPACE: O(n)
```
</details>

<details>
<summary> 79. Word Search </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        int rows = board.size(), cols = board[0].size();
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                if (dfs(board, word, 0, i, j)) return true;
            }
        }
        return false;
    }
    
    bool dfs(vector<vector<char>>& board, string& word, int index, int i, int j) {
        if (index == word.length()) return true;
        if (i < 0 || i >= board.size() || j < 0 || j >= board[0].size()) return false;
        if (board[i][j] != word[index]) return false;
        char temp = board[i][j];
        board[i][j] = '#';
        bool found = dfs(board, word, index + 1, i + 1, j) ||
                     dfs(board, word, index + 1, i - 1, j) ||
                     dfs(board, word, index + 1, i, j + 1) ||
                     dfs(board, word, index + 1, i, j - 1);
        board[i][j] = temp;
        return found;
    }
};

// TIME: O(m * n * 4^L) where L is word length
// SPACE: O(L)
```
</details>

<details>
<summary> 131. Palindrome Partitioning </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<vector<string>> partition(string s) {
        vector<vector<string>> result;
        vector<string> current;
        backtrack(s, 0, current, result);
        return result;
    }
    
    void backtrack(string& s, int start, vector<string>& current,
                   vector<vector<string>>& result) {
        if (start == s.length()) {
            result.push_back(current);
            return;
        }
        for (int end = start + 1; end <= s.length(); ++end) {
            if (isPalindrome(s, start, end - 1)) {
                current.push_back(s.substr(start, end - start));
                backtrack(s, end, current, result);
                current.pop_back();
            }
        }
    }
    
    bool isPalindrome(string& s, int left, int right) {
        while (left < right) {
            if (s[left++] != s[right--]) return false;
        }
        return true;
    }
};

// TIME: O(n * 2^n)
// SPACE: O(n)
```
</details>

<details>
<summary> 51. N-Queens </summary>

```cpp
#include <vector>
#include <string>
#include <unordered_set>
using namespace std;

class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> result;
        vector<string> board(n, string(n, '.'));
        unordered_set<int> cols, diag1, diag2;
        backtrack(0, n, board, cols, diag1, diag2, result);
        return result;
    }
    
    void backtrack(int row, int n, vector<string>& board,
                   unordered_set<int>& cols, unordered_set<int>& diag1,
                   unordered_set<int>& diag2, vector<vector<string>>& result) {
        if (row == n) {
            result.push_back(board);
            return;
        }
        for (int col = 0; col < n; ++col) {
            int d1 = row - col;
            int d2 = row + col;
            if (cols.count(col) || diag1.count(d1) || diag2.count(d2)) continue;
            board[row][col] = 'Q';
            cols.insert(col);
            diag1.insert(d1);
            diag2.insert(d2);
            backtrack(row + 1, n, board, cols, diag1, diag2, result);
            board[row][col] = '.';
            cols.erase(col);
            diag1.erase(d1);
            diag2.erase(d2);
        }
    }
};

// TIME: O(n!) 
// SPACE: O(n)
```
</details>


### 🔎 10. Binary Search

**Key Techniques**: On sorted arrays, reduce search range by half. Handle boundaries carefully (`left <= right` vs `left < right`), and manage mid calculation to avoid overflow: `mid = left + (right-left)/2`.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 34 | **[Find First and Last Position in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-sorted-array/)** | Two binary searches: first find left boundary, then right boundary. | [C++ Solution](#find-first-and-last-position) | Medium |
| 33 | **[Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)** | Identify which half is sorted, then narrow search accordingly. | [C++ Solution](#search-in-rotated-sorted-array) | Medium |
| 153 | **[Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)** | Binary search: if `nums[mid] < nums[right]`, minimum is in left half (including mid); else in right half. | [C++ Solution](#find-minimum-in-rotated-sorted-array) | Medium |
| 4 | **[Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)** | Binary search on smaller array to find partition point such that left elements <= right elements. | [C++ Solution](#median-of-two-sorted-arrays) | Hard |
| 35 | **[Search Insert Position](https://leetcode.com/problems/search-insert-position/)** | Standard binary search, return `left` when `target` not found. | [C++ Solution](#search-insert-position) | Easy |
| 74 | **[Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)** | Treat matrix as 1D by index mapping: `row = mid / n`, `col = mid % n`. | [C++ Solution](#search-2d-matrix) | Medium |

#### C++ Code Examples


<details>
<summary> 34. Find First and Last Position in Sorted Array </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int left = findBound(nums, target, true);
        int right = findBound(nums, target, false);
        return {left, right};
    }
    
    int findBound(vector<int>& nums, int target, bool isLeft) {
        int left = 0, right = nums.size() - 1;
        int bound = -1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                bound = mid;
                if (isLeft) right = mid - 1;
                else left = mid + 1;
            } else if (nums[mid] < target) left = mid + 1;
            else right = mid - 1;
        }
        return bound;
    }
};

// TIME: O(log n)
// SPACE: O(1)
```
</details>

<details>
<summary> 33. Search in Rotated Sorted Array </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0, right = nums.size() - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;
            // left half is sorted
            if (nums[left] <= nums[mid]) {
                if (nums[left] <= target && target < nums[mid]) right = mid - 1;
                else left = mid + 1;
            } else { // right half is sorted
                if (nums[mid] < target && target <= nums[right]) left = mid + 1;
                else right = mid - 1;
            }
        }
        return -1;
    }
};

// TIME: O(log n)
// SPACE: O(1)
```
</details>

<details>
<summary> 153. Find Minimum in Rotated Sorted Array </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int findMin(vector<int>& nums) {
        int left = 0, right = nums.size() - 1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < nums[right]) right = mid;
            else left = mid + 1;
        }
        return nums[left];
    }
};

// TIME: O(log n)
// SPACE: O(1)
```
</details>

<details>
<summary> 4. Median of Two Sorted Arrays </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        if (nums1.size() > nums2.size()) return findMedianSortedArrays(nums2, nums1);
        int m = nums1.size(), n = nums2.size();
        int left = 0, right = m;
        while (left <= right) {
            int partition1 = left + (right - left) / 2;
            int partition2 = (m + n + 1) / 2 - partition1;
            int maxLeft1 = (partition1 == 0) ? INT_MIN : nums1[partition1 - 1];
            int minRight1 = (partition1 == m) ? INT_MAX : nums1[partition1];
            int maxLeft2 = (partition2 == 0) ? INT_MIN : nums2[partition2 - 1];
            int minRight2 = (partition2 == n) ? INT_MAX : nums2[partition2];
            if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
                if ((m + n) % 2 == 0) return (max(maxLeft1, maxLeft2) + min(minRight1, minRight2)) / 2.0;
                else return max(maxLeft1, maxLeft2);
            } else if (maxLeft1 > minRight2) right = partition1 - 1;
            else left = partition1 + 1;
        }
        return 0.0;
    }
};

// TIME: O(log(min(m, n)))
// SPACE: O(1)
```
</details>

<details>
<summary> 35. Search Insert Position </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0, right = nums.size() - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) return mid;
            else if (nums[mid] < target) left = mid + 1;
            else right = mid - 1;
        }
        return left;
    }
};

// TIME: O(log n)
// SPACE: O(1)
```
</details>

<details>
<summary> 74. Search a 2D Matrix </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if (matrix.empty() || matrix[0].empty()) return false;
        int m = matrix.size(), n = matrix[0].size();
        int left = 0, right = m * n - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            int val = matrix[mid / n][mid % n];
            if (val == target) return true;
            else if (val < target) left = mid + 1;
            else right = mid - 1;
        }
        return false;
    }
};

// TIME: O(log(m * n))
// SPACE: O(1)
```
</details>


### 📚 11. Stack & Queue

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 20 | **[Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)** | Use stack: push opening brackets, when closing bracket is found, check if top matches. | [C++ Solution](#valid-parentheses) | Easy |
| 155 | **[Min Stack](https://leetcode.com/problems/min-stack/)** | Use two stacks: one for actual values, one for current minimums. | [C++ Solution](#min-stack) | Medium |
| 394 | **[Decode String](https://leetcode.com/problems/decode-string/)** | Use two stacks: one for counts, one for strings. Append chars until encountering ']', then pop. | [C++ Solution](#decode-string) | Medium |
| 739 | **[Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)** | Monotonic decreasing stack: store indices, when warmer found, pop and record days difference. | [C++ Solution](#daily-temperatures) | Medium |
| 84 | **[Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)** | Monotonic stack: for each bar, find left and right smaller boundaries using stack. Compute area = height * (width). | [C++ Solution](#largest-rectangle-in-histogram) | Hard |
| 496 | **[Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/)** | Use monotonic stack and hash map: for each element, find next greater, store in map. | [C++ Solution](#next-greater-element) | Easy |

#### C++ Code Examples


<details>
<summary> 20. Valid Parentheses </summary>

```cpp
#include <string>
#include <stack>
#include <unordered_map>
using namespace std;

class Solution {
public:
    bool isValid(string s) {
        stack<char> stk;
        unordered_map<char, char> matching = {{')', '('}, {']', '['}, {'}', '{'}};
        for (char c : s) {
            if (matching.find(c) != matching.end()) {
                if (stk.empty() || stk.top() != matching[c]) return false;
                stk.pop();
            } else stk.push(c);
        }
        return stk.empty();
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 155. Min Stack </summary>

```cpp
#include <stack>
using namespace std;

class MinStack {
private:
    stack<int> stk;
    stack<int> minStk;
public:
    MinStack() {}
    
    void push(int val) {
        stk.push(val);
        if (minStk.empty() || val <= minStk.top()) minStk.push(val);
    }
    
    void pop() {
        if (stk.top() == minStk.top()) minStk.pop();
        stk.pop();
    }
    
    int top() { return stk.top(); }
    int getMin() { return minStk.top(); }
};

// TIME: O(1) for all operations
// SPACE: O(n)
```
</details>

<details>
<summary> 394. Decode String </summary>

```cpp
#include <string>
#include <stack>
using namespace std;

class Solution {
public:
    string decodeString(string s) {
        stack<int> countStk;
        stack<string> stringStk;
        string current = "";
        int k = 0;
        for (char c : s) {
            if (isdigit(c)) k = k * 10 + (c - '0');
            else if (isalpha(c)) current += c;
            else if (c == '[') {
                countStk.push(k);
                stringStk.push(current);
                current = "";
                k = 0;
            } else if (c == ']') {
                string decoded = stringStk.top();
                stringStk.pop();
                int repeat = countStk.top();
                countStk.pop();
                for (int i = 0; i < repeat; ++i) decoded += current;
                current = decoded;
            }
        }
        return current;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 739. Daily Temperatures </summary>

```cpp
#include <vector>
#include <stack>
using namespace std;

class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        vector<int> result(n, 0);
        stack<int> stk;  // indices
        for (int i = 0; i < n; ++i) {
            while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
                int idx = stk.top();
                stk.pop();
                result[idx] = i - idx;
            }
            stk.push(i);
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 84. Largest Rectangle in Histogram </summary>

```cpp
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int> stk;
        heights.push_back(0);
        int maxArea = 0;
        for (int i = 0; i < heights.size(); ++i) {
            while (!stk.empty() && heights[i] < heights[stk.top()]) {
                int h = heights[stk.top()];
                stk.pop();
                int width = stk.empty() ? i : i - stk.top() - 1;
                maxArea = max(maxArea, h * width);
            }
            stk.push(i);
        }
        heights.pop_back();
        return maxArea;
    }
};

// TIME: O(n)
// SPACE: O(n)
```
</details>

<details>
<summary> 496. Next Greater Element I </summary>

```cpp
#include <vector>
#include <stack>
#include <unordered_map>
using namespace std;

class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int> nextGreater;
        stack<int> stk;
        for (int num : nums2) {
            while (!stk.empty() && stk.top() < num) {
                nextGreater[stk.top()] = num;
                stk.pop();
            }
            stk.push(num);
        }
        vector<int> result;
        for (int num : nums1) result.push_back(nextGreater.count(num) ? nextGreater[num] : -1);
        return result;
    }
};

// TIME: O(m + n)
// SPACE: O(n)
```
</details>


### ⛰️ 12. Heap / Priority Queue

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 215 | **[Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)** | Use binary min-heap of size `k` (push, pop if size > k). After processing, top is kth largest. | [C++ Solution](#kth-largest-element) | Medium |
| 347 | **[Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)** | Use hash map to count frequencies. Use min-heap of size `k` to keep top k frequent elements by frequency. | [C++ Solution](#top-k-frequent-elements) | Medium |
| 23 | **[Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)** | Use min-heap (priority queue) to always select the smallest among k heads. | [C++ Solution](#merge-k-sorted-lists-heap) | Hard |
| 295 | **[Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)** | Use two heaps: max-heap for left half, min-heap for right half. Maintain balance. | [C++ Solution](#find-median-from-data-stream) | Hard |
| 1046 | **[Last Stone Weight](https://leetcode.com/problems/last-stone-weight/)** | Put all stones in max-heap. While >1 stone, pop two largest, if different push difference back. | [C++ Solution](#last-stone-weight) | Easy |

#### C++ Code Examples


<details>
<summary> 215. Kth Largest Element in an Array </summary>

```cpp
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        priority_queue<int, vector<int>, greater<int>> minHeap;
        for (int num : nums) {
            minHeap.push(num);
            if (minHeap.size() > k) minHeap.pop();
        }
        return minHeap.top();
    }
};

// TIME: O(n log k)
// SPACE: O(k)
```
</details>

<details>
<summary> 347. Top K Frequent Elements </summary>

```cpp
#include <vector>
#include <unordered_map>
#include <queue>
using namespace std;

class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> freq;
        for (int num : nums) freq[num]++;
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> minHeap;
        for (auto& p : freq) {
            minHeap.push({p.second, p.first});
            if (minHeap.size() > k) minHeap.pop();
        }
        vector<int> result;
        while (!minHeap.empty()) {
            result.push_back(minHeap.top().second);
            minHeap.pop();
        }
        return result;
    }
};

// TIME: O(n log k)
// SPACE: O(n)
```
</details>

<details>
<summary> 23. Merge k Sorted Lists (Heap) </summary>

```cpp
#include <vector>
#include <queue>
#include <cstddef>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

struct Compare {
    bool operator()(ListNode* a, ListNode* b) { return a->val > b->val; }
};

class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*, vector<ListNode*>, Compare> pq;
        for (ListNode* list : lists) {
            if (list) pq.push(list);
        }
        ListNode dummy;
        ListNode* tail = &dummy;
        while (!pq.empty()) {
            ListNode* smallest = pq.top();
            pq.pop();
            tail->next = smallest;
            tail = tail->next;
            if (smallest->next) pq.push(smallest->next);
        }
        return dummy.next;
    }
};

// TIME: O(n log k)
// SPACE: O(k)
```
</details>

<details>
<summary> 295. Find Median from Data Stream </summary>

```cpp
#include <queue>
using namespace std;

class MedianFinder {
private:
    priority_queue<int> maxHeap;  // left half
    priority_queue<int, vector<int>, greater<int>> minHeap;  // right half
public:
    MedianFinder() {}
    
    void addNum(int num) {
        if (maxHeap.empty() || num <= maxHeap.top()) maxHeap.push(num);
        else minHeap.push(num);
        // balance heaps so maxHeap size is >= minHeap size
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        } else if (minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }
    
    double findMedian() {
        if (maxHeap.size() == minHeap.size()) return (maxHeap.top() + minHeap.top()) / 2.0;
        return maxHeap.top();
    }
};

// TIME: O(log n) for addNum, O(1) for findMedian
// SPACE: O(n)
```
</details>

<details>
<summary> 1046. Last Stone Weight </summary>

```cpp
#include <vector>
#include <queue>
#include <cmath>
using namespace std;

class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        priority_queue<int> maxHeap(stones.begin(), stones.end());
        while (maxHeap.size() > 1) {
            int a = maxHeap.top(); maxHeap.pop();
            int b = maxHeap.top(); maxHeap.pop();
            if (a != b) maxHeap.push(a - b);
        }
        return maxHeap.empty() ? 0 : maxHeap.top();
    }
};

// TIME: O(n log n)
// SPACE: O(n)
```
</details>


### 👤 13. Greedy

**Key Techniques**: Make locally optimal choices that lead to globally optimal solution. Often used for interval scheduling.

| # | Problem | Solution Approach | Code in C++ | Difficulty |
|---|---------|-------------------|-------------|------------|
| 55 | **[Jump Game](https://leetcode.com/problems/jump-game/)** | Track `maxReachable`. For each index `i`, if `i > maxReachable` return false. Update `maxReachable = max(maxReachable, i + nums[i])`. | [C++ Solution](#jump-game) | Medium |
| 45 | **[Jump Game II](https://leetcode.com/problems/jump-game-ii/)** | Use BFS-style approach: track `jumps`, `currentEnd`, `farthest`. For each i, update farthest, when reaching currentEnd, increment jumps and set currentEnd = farthest. | [C++ Solution](#jump-game-ii) | Medium |
| 435 | **[Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)** | Sort intervals by end time. Greedy: pick intervals that end earliest, count overlaps. | [C++ Solution](#non-overlapping-intervals) | Medium |
| 763 | **[Partition Labels](https://leetcode.com/problems/partition-labels/)** | Record last occurrence of each character. Iterate, keep current partition's `last` index. When `i == last`, make partition. | [C++ Solution](#partition-labels) | Medium |
| 134 | **[Gas Station](https://leetcode.com/problems/gas-station/)** | If total gas < total cost → impossible. Find starting index where running total never negative. | [C++ Solution](#gas-station) | Medium |

#### C++ Code Examples


<details>
<summary> 55. Jump Game </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maxReachable = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (i > maxReachable) return false;
            maxReachable = max(maxReachable, i + nums[i]);
            if (maxReachable >= nums.size() - 1) return true;
        }
        return true;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 45. Jump Game II </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int jump(vector<int>& nums) {
        int jumps = 0, currentEnd = 0, farthest = 0;
        for (int i = 0; i < nums.size() - 1; ++i) {
            farthest = max(farthest, i + nums[i]);
            if (i == currentEnd) {
                jumps++;
                currentEnd = farthest;
                if (currentEnd >= nums.size() - 1) break;
            }
        }
        return jumps;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 435. Non-overlapping Intervals </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(),
             [](const vector<int>& a, const vector<int>& b) { return a[1] < b[1]; });
        int count = 0;
        int lastEnd = intervals[0][1];
        for (int i = 1; i < intervals.size(); ++i) {
            if (intervals[i][0] < lastEnd) count++;
            else lastEnd = intervals[i][1];
        }
        return count;
    }
};

// TIME: O(n log n)
// SPACE: O(1)
```
</details>

<details>
<summary> 763. Partition Labels </summary>

```cpp
#include <vector>
#include <string>
using namespace std;

class Solution {
public:
    vector<int> partitionLabels(string s) {
        vector<int> lastOccurrence(26, -1);
        for (int i = 0; i < s.length(); ++i) lastOccurrence[s[i] - 'a'] = i;
        vector<int> result;
        int start = 0, last = 0;
        for (int i = 0; i < s.length(); ++i) {
            last = max(last, lastOccurrence[s[i] - 'a']);
            if (i == last) {
                result.push_back(last - start + 1);
                start = i + 1;
            }
        }
        return result;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 134. Gas Station </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int total = 0, current = 0, start = 0;
        for (int i = 0; i < gas.size(); ++i) {
            total += gas[i] - cost[i];
            current += gas[i] - cost[i];
            if (current < 0) {
                current = 0;
                start = i + 1;
            }
        }
        return total >= 0 ? start : -1;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>


### 🧠 14. Dynamic Programming

**Key Techniques**: Recognize overlapping sub-problems and optimal substructure. Design `dp[i]` meaning (e.g., max sum ending at i). Define recurrence relation. Use tabulation (bottom-up) or memoization (top-down). Start with simple brute force recursion, then add memoization for optimization.

| # | Problem | Solution Approach | C++ Solution Example | Difficulty |
|---|---------|-------------------|----------------------|------------|
| 53 | **[Maximum Subarray (Kadane's Algorithm)](https://leetcode.com/problems/maximum-subarray/)** | `dp[i] = max(nums[i], dp[i - 1] + nums[i])`. Also covered in array section. | [C++ Solution](#maximum-subarray-again) | Easy |
| 70 | **[Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)** | Fibonacci style: `dp[i] = dp[i - 1] + dp[i - 2]`. | [C++ Solution](#climbing-stairs) | Easy |
| 198 | **[House Robber](https://leetcode.com/problems/house-robber/)** | `dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])`. | [C++ Solution](#house-robber) | Medium |
| 213 | **[House Robber II](https://leetcode.com/problems/house-robber-ii/)** | Solve two cases: rob without first house, rob without last house. | [C++ Solution](#house-robber-ii) | Medium |
| 5 | **[Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)** | Expand around center approach O(n²). | [C++ Solution](#longest-palindromic-substring) | Medium |
| 322 | **[Coin Change](https://leetcode.com/problems/coin-change/)** | `dp[i] = min(dp[i], dp[i - coin] + 1)`. | [C++ Solution](#coin-change) | Medium |
| 300 | **[Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)** | DP O(n²) or patience sorting with binary search O(n log n). | [C++ Solution](#longest-increasing-subsequence) | Medium |
| 1143 | **[Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)** | 2D DP: if `text1[i-1] == text2[j-1]` then `dp[i][j] = 1 + dp[i-1][j-1]`, else `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`. | [C++ Solution](#longest-common-subsequence) | Medium |
| 139 | **[Word Break](https://leetcode.com/problems/word-break/)** | DP: `dp[i] = true` if `dp[j]` is true and `s[j:i]` in word set. | [C++ Solution](#word-break) | Medium |

#### C++ Code Examples


<details>
<summary> 53. Maximum Subarray (Kadane's Algorithm) </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum = nums[0];
        int currentSum = nums[0];
        for (int i = 1; i < nums.size(); ++i) {
            currentSum = max(nums[i], currentSum + nums[i]);
            maxSum = max(maxSum, currentSum);
        }
        return maxSum;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 70. Climbing Stairs </summary>

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    int climbStairs(int n) {
        if (n <= 2) return n;
        int prev2 = 1, prev1 = 2;
        for (int i = 3; i <= n; ++i) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }
        return prev1;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 198. House Robber </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.empty()) return 0;
        if (nums.size() == 1) return nums[0];
        int dp0 = nums[0];
        int dp1 = max(nums[0], nums[1]);
        for (int i = 2; i < nums.size(); ++i) {
            int current = max(dp1, dp0 + nums[i]);
            dp0 = dp1;
            dp1 = current;
        }
        return dp1;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 213. House Robber II </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.empty()) return 0;
        if (nums.size() == 1) return nums[0];
        return max(robHelper(nums, 0, nums.size() - 2),
                   robHelper(nums, 1, nums.size() - 1));
    }
    
    int robHelper(vector<int>& nums, int start, int end) {
        int dp0 = 0, dp1 = 0;
        for (int i = start; i <= end; ++i) {
            int current = max(dp1, dp0 + nums[i]);
            dp0 = dp1;
            dp1 = current;
        }
        return dp1;
    }
};

// TIME: O(n)
// SPACE: O(1)
```
</details>

<details>
<summary> 5. Longest Palindromic Substring </summary>

```cpp
#include <string>
using namespace std;

class Solution {
public:
    string longestPalindrome(string s) {
        if (s.empty()) return "";
        int start = 0, maxLen = 1;
        for (int i = 0; i < s.length(); ++i) {
            expandAroundCenter(s, i, i, start, maxLen);
            expandAroundCenter(s, i, i + 1, start, maxLen);
        }
        return s.substr(start, maxLen);
    }
    
    void expandAroundCenter(const string& s, int left, int right, int& start, int& maxLen) {
        while (left >= 0 && right < s.length() && s[left] == s[right]) {
            int len = right - left + 1;
            if (len > maxLen) {
                maxLen = len;
                start = left;
            }
            --left; ++right;
        }
    }
};

// TIME: O(n²)
// SPACE: O(1)
```
</details>

<details>
<summary> 322. Coin Change </summary>

```cpp
#include <vector>
#include <climits>
using namespace std;

class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);
        dp[0] = 0;
        for (int i = 1; i <= amount; ++i) {
            for (int coin : coins) {
                if (i - coin >= 0) dp[i] = min(dp[i], dp[i - coin] + 1);
            }
        }
        return dp[amount] == amount + 1 ? -1 : dp[amount];
    }
};

// TIME: O(amount * n)
// SPACE: O(amount)
```
</details>

<details>
<summary> 300. Longest Increasing Subsequence </summary>

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> tails;
        for (int num : nums) {
            auto it = lower_bound(tails.begin(), tails.end(), num);
            if (it == tails.end()) tails.push_back(num);
            else *it = num;
        }
        return tails.size();
    }
};

// TIME: O(n log n)
// SPACE: O(n)
```
</details>

<details>
<summary> 1143. Longest Common Subsequence </summary>

```cpp
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.length(), n = text2.length();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (text1[i - 1] == text2[j - 1]) dp[i][j] = 1 + dp[i - 1][j - 1];
                else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[m][n];
    }
};

// TIME: O(m * n)
// SPACE: O(m * n)
```
</details>

<details>
<summary> 139. Word Break </summary>

```cpp
#include <vector>
#include <string>
#include <unordered_set>
using namespace std;

class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> wordSet(wordDict.begin(), wordDict.end());
        vector<bool> dp(s.length() + 1, false);
        dp[0] = true;
        for (int i = 1; i <= s.length(); ++i) {
            for (int j = 0; j < i; ++j) {
                if (dp[j] && wordSet.count(s.substr(j, i - j))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
};

// TIME: O(n³)
// SPACE: O(n)
```
</details>


### 💡 Interview Tips & Common Pitfalls

*   **Always Ask Clarifying Questions**: Is the array sorted? Can I use extra space? What are the input constraints?
*   **Think Out Loud**: Explain your reasoning as you code. Interviewers want to see your thought process.
*   **Test Edge Cases**: Empty inputs, single elements, large values, duplicates.
*   **Time vs. Space Trade-offs**: Sometimes brute force is acceptable as a starting point, then optimize.
*   **Debug with Simple Examples**: Mentally run through small test cases to catch off-by-one errors.

Best of luck with your interview preparation!

If you'd like me to elaborate on any specific problem or concept, please feel free to ask.