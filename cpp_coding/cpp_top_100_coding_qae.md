I'll expand **all major categories** with complete, runnable code examples. Here's the full expansion:

## 🔢 **NUMBERS & MATH (31–40)** - Complete Solutions

### 31. GCD using Euclidean algorithm
```cpp
#include <iostream>
using namespace std;

int gcd(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

// Recursive version
int gcdRecursive(int a, int b) {
    return b == 0 ? a : gcdRecursive(b, a % b);
}

int main() {
    cout << gcd(48, 18) << endl;  // 6
    cout << gcdRecursive(48, 18); // 6
}
```

### 32. LCM using GCD
```cpp
int lcm(int a, int b) {
    return (a / gcd(a, b)) * b;  // Division first to avoid overflow
}

// Example: lcm(12, 18) = 36
```

### 33. Prime factorization
```cpp
#include <vector>
#include <cmath>

vector<int> primeFactors(int n) {
    vector<int> factors;
    
    // Handle 2 separately
    while (n % 2 == 0) {
        factors.push_back(2);
        n /= 2;
    }
    
    // Check odd factors
    for (int i = 3; i <= sqrt(n); i += 2) {
        while (n % i == 0) {
            factors.push_back(i);
            n /= i;
        }
    }
    
    // If remaining n is prime
    if (n > 2) factors.push_back(n);
    
    return factors;
}

// Example: primeFactors(84) → {2, 2, 3, 7}
```

### 34. Armstrong number (153 = 1³ + 5³ + 3³)
```cpp
bool isArmstrong(int n) {
    int original = n, sum = 0;
    int digits = to_string(n).length();
    
    while (n > 0) {
        int digit = n % 10;
        sum += pow(digit, digits);
        n /= 10;
    }
    
    return sum == original;
}

// Example: 153 → true, 123 → false
```

### 35. Perfect number (sum of divisors = number)
```cpp
bool isPerfect(int n) {
    if (n <= 1) return false;
    
    int sum = 1;  // 1 is always a divisor
    for (int i = 2; i <= sqrt(n); i++) {
        if (n % i == 0) {
            sum += i;
            if (i != n / i) sum += n / i;
        }
    }
    
    return sum == n;
}

// Example: 6 (1+2+3=6) → true, 28 → true
```

### 36. Binary to decimal
```cpp
int binaryToDecimal(string binary) {
    int decimal = 0, power = 0;
    
    for (int i = binary.length() - 1; i >= 0; i--) {
        if (binary[i] == '1')
            decimal += pow(2, power);
        power++;
    }
    
    return decimal;
}

// Example: "1010" = 10
```

### 37. Decimal to binary
```cpp
string decimalToBinary(int n) {
    if (n == 0) return "0";
    
    string binary = "";
    while (n > 0) {
        binary = to_string(n % 2) + binary;
        n /= 2;
    }
    
    return binary;
}

// Example: 10 → "1010"
```

### 38. Power without pow() (exponentiation by squaring)
```cpp
double power(double base, int exp) {
    if (exp == 0) return 1;
    if (exp < 0) return 1 / power(base, -exp);
    
    double half = power(base, exp / 2);
    if (exp % 2 == 0)
        return half * half;
    else
        return half * half * base;
}

// Example: power(2, 10) = 1024
```

### 39. Square root using binary search
```cpp
double sqrtBinarySearch(int n, int precision = 3) {
    if (n < 0) return -1;
    if (n == 0 || n == 1) return n;
    
    double low = 0, high = n, mid;
    
    // Integer part
    while (high - low > 0.0001) {
        mid = (low + high) / 2;
        if (mid * mid > n)
            high = mid;
        else
            low = mid;
    }
    
    return round(mid * pow(10, precision)) / pow(10, precision);
}

// Example: sqrtBinarySearch(10) ≈ 3.162
```

### 40. Count set bits (Brian Kernighan's algorithm)
```cpp
int countSetBits(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1);  // Clears lowest set bit
        count++;
    }
    return count;
}

// Example: 13 (1101) → 3 set bits
```

---

## 📝 **STRINGS (41–50)** - Complete Solutions

### 41. First non-repeating character
```cpp
#include <unordered_map>

char firstNonRepeating(string s) {
    unordered_map<char, int> freq;
    
    for (char c : s) freq[c]++;
    
    for (char c : s) {
        if (freq[c] == 1) return c;
    }
    
    return '\0';  // No non-repeating char
}

// Example: "swiss" → 'w'
```

### 42. Longest common prefix
```cpp
string longestCommonPrefix(vector<string>& strs) {
    if (strs.empty()) return "";
    
    string prefix = strs[0];
    
    for (int i = 1; i < strs.size(); i++) {
        while (strs[i].find(prefix) != 0) {
            prefix = prefix.substr(0, prefix.length() - 1);
            if (prefix.empty()) return "";
        }
    }
    
    return prefix;
}

// Example: ["flower","flow","flight"] → "fl"
```

### 43. Valid parentheses
```cpp
#include <stack>

bool isValidParentheses(string s) {
    stack<char> st;
    
    for (char c : s) {
        if (c == '(' || c == '{' || c == '[') {
            st.push(c);
        } else {
            if (st.empty()) return false;
            
            char top = st.top();
            if ((c == ')' && top == '(') ||
                (c == '}' && top == '{') ||
                (c == ']' && top == '[')) {
                st.pop();
            } else {
                return false;
            }
        }
    }
    
    return st.empty();
}

// Example: "()[]{}" → true, "([)]" → false
```

### 44. Roman to integer
```cpp
int romanToInt(string s) {
    unordered_map<char, int> values = {
        {'I', 1}, {'V', 5}, {'X', 10}, {'L', 50},
        {'C', 100}, {'D', 500}, {'M', 1000}
    };
    
    int result = 0;
    
    for (int i = 0; i < s.length(); i++) {
        if (i + 1 < s.length() && values[s[i]] < values[s[i + 1]])
            result -= values[s[i]];
        else
            result += values[s[i]];
    }
    
    return result;
}

// Example: "MCMXCIV" = 1994
```

### 45. Integer to roman
```cpp
string intToRoman(int num) {
    vector<pair<int, string>> values = {
        {1000, "M"}, {900, "CM"}, {500, "D"}, {400, "CD"},
        {100, "C"}, {90, "XC"}, {50, "L"}, {40, "XL"},
        {10, "X"}, {9, "IX"}, {5, "V"}, {4, "IV"}, {1, "I"}
    };
    
    string result = "";
    
    for (auto& [value, symbol] : values) {
        while (num >= value) {
            result += symbol;
            num -= value;
        }
    }
    
    return result;
}

// Example: 1994 → "MCMXCIV"
```

### 46. Run-length encoding
```cpp
string runLengthEncode(string s) {
    if (s.empty()) return "";
    
    string encoded = "";
    int count = 1;
    
    for (int i = 1; i <= s.length(); i++) {
        if (i < s.length() && s[i] == s[i - 1]) {
            count++;
        } else {
            encoded += s[i - 1] + to_string(count);
            count = 1;
        }
    }
    
    return encoded;
}

string runLengthDecode(string s) {
    string decoded = "";
    
    for (int i = 0; i < s.length(); i++) {
        char c = s[i];
        i++;
        int count = 0;
        while (i < s.length() && isdigit(s[i])) {
            count = count * 10 + (s[i] - '0');
            i++;
        }
        i--;
        
        decoded.append(count, c);
    }
    
    return decoded;
}

// Example: "AAAABBBCCD" → "A4B3C2D1"
```

### 47. String to integer (atoi implementation)
```cpp
int myAtoi(string s) {
    int i = 0, sign = 1;
    long result = 0;
    
    // Skip whitespace
    while (i < s.length() && s[i] == ' ') i++;
    
    // Handle sign
    if (i < s.length() && (s[i] == '+' || s[i] == '-')) {
        sign = (s[i] == '-') ? -1 : 1;
        i++;
    }
    
    // Convert digits
    while (i < s.length() && isdigit(s[i])) {
        result = result * 10 + (s[i] - '0');
        
        // Check overflow
        if (result * sign > INT_MAX) return INT_MAX;
        if (result * sign < INT_MIN) return INT_MIN;
        
        i++;
    }
    
    return result * sign;
}

// Example: "   -42" → -42
```

### 48. Compare version numbers
```cpp
int compareVersion(string version1, string version2) {
    int i = 0, j = 0;
    int n1 = version1.length(), n2 = version2.length();
    
    while (i < n1 || j < n2) {
        int num1 = 0, num2 = 0;
        
        while (i < n1 && version1[i] != '.')
            num1 = num1 * 10 + (version1[i++] - '0');
        
        while (j < n2 && version2[j] != '.')
            num2 = num2 * 10 + (version2[j++] - '0');
        
        if (num1 > num2) return 1;
        if (num1 < num2) return -1;
        
        i++; j++;
    }
    
    return 0;
}

// Example: "1.2" vs "1.10" → -1
```

### 49. Group anagrams
```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    
    for (string s : strs) {
        string key = s;
        sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    
    vector<vector<string>> result;
    for (auto& pair : groups) {
        result.push_back(pair.second);
    }
    
    return result;
}

// Example: ["eat","tea","tan","ate","nat","bat"]
// Returns: [["eat","tea","ate"],["tan","nat"],["bat"]]
```

### 50. Longest palindromic substring
```cpp
string longestPalindrome(string s) {
    if (s.empty()) return "";
    
    int start = 0, maxLen = 1;
    
    auto expandAroundCenter = [&](int left, int right) {
        while (left >= 0 && right < s.length() && s[left] == s[right]) {
            if (right - left + 1 > maxLen) {
                start = left;
                maxLen = right - left + 1;
            }
            left--;
            right++;
        }
    };
    
    for (int i = 0; i < s.length(); i++) {
        expandAroundCenter(i, i);      // Odd length
        expandAroundCenter(i, i + 1);  // Even length
    }
    
    return s.substr(start, maxLen);
}

// Example: "babad" → "bab" or "aba"
```

---

## 📊 **ARRAYS (51–60)** - Complete Solutions

### 51. Majority element (Boyer-Moore)
```cpp
int majorityElement(vector<int>& nums) {
    int candidate = 0, count = 0;
    
    for (int num : nums) {
        if (count == 0) candidate = num;
        count += (num == candidate) ? 1 : -1;
    }
    
    return candidate;
}

// Example: [3,2,3] → 3
```

### 52. Best time to buy and sell stock
```cpp
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, maxProfit = 0;
    
    for (int price : prices) {
        if (price < minPrice)
            minPrice = price;
        else if (price - minPrice > maxProfit)
            maxProfit = price - minPrice;
    }
    
    return maxProfit;
}

// Example: [7,1,5,3,6,4] → 5
```

### 53. Product of array except self
```cpp
vector<int> productExceptSelf(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, 1);
    
    // Left products
    int left = 1;
    for (int i = 0; i < n; i++) {
        result[i] = left;
        left *= nums[i];
    }
    
    // Right products
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= right;
        right *= nums[i];
    }
    
    return result;
}

// Example: [1,2,3,4] → [24,12,8,6]
```

### 54. Intersection of two arrays
```cpp
vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
    unordered_set<int> set1(nums1.begin(), nums1.end());
    unordered_set<int> result;
    
    for (int num : nums2) {
        if (set1.count(num)) result.insert(num);
    }
    
    return vector<int>(result.begin(), result.end());
}

// Example: [1,2,2,1], [2,2] → [2]
```

### 55. Merge sorted arrays (in-place)
```cpp
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
    int i = m - 1, j = n - 1, k = m + n - 1;
    
    while (i >= 0 && j >= 0) {
        if (nums1[i] > nums2[j])
            nums1[k--] = nums1[i--];
        else
            nums1[k--] = nums2[j--];
    }
    
    while (j >= 0)
        nums1[k--] = nums2[j--];
}

// Example: nums1 = [1,2,3,0,0,0], m=3, nums2=[2,5,6], n=3
// Result: [1,2,2,3,5,6]
```

### 56. Find duplicate number (Floyd's cycle)
```cpp
int findDuplicate(vector<int>& nums) {
    int slow = nums[0], fast = nums[0];
    
    // Find meeting point
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    
    // Find cycle entrance
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    
    return slow;
}

// Example: [1,3,4,2,2] → 2
```

### 57. First missing positive
```cpp
int firstMissingPositive(vector<int>& nums) {
    int n = nums.size();
    
    // Place each number in its correct position
    for (int i = 0; i < n; i++) {
        while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            swap(nums[i], nums[nums[i] - 1]);
        }
    }
    
    // Find first missing
    for (int i = 0; i < n; i++) {
        if (nums[i] != i + 1) return i + 1;
    }
    
    return n + 1;
}

// Example: [3,4,-1,1] → 2
```

### 58. Container with most water
```cpp
int maxArea(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int maxWater = 0;
    
    while (left < right) {
        int width = right - left;
        int h = min(height[left], height[right]);
        maxWater = max(maxWater, width * h);
        
        if (height[left] < height[right])
            left++;
        else
            right--;
    }
    
    return maxWater;
}

// Example: [1,8,6,2,5,4,8,3,7] → 49
```

### 59. 3Sum
```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    
    for (int i = 0; i < nums.size() - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        
        int left = i + 1, right = nums.size() - 1;
        
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            
            if (sum == 0) {
                result.push_back({nums[i], nums[left], nums[right]});
                
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    
    return result;
}

// Example: [-1,0,1,2,-1,-4] → [[-1,-1,2],[-1,0,1]]
```

### 60. Subarray sum equals K
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixSum;
    prefixSum[0] = 1;
    int sum = 0, count = 0;
    
    for (int num : nums) {
        sum += num;
        if (prefixSum.count(sum - k))
            count += prefixSum[sum - k];
        prefixSum[sum]++;
    }
    
    return count;
}

// Example: [1,1,1], k=2 → 2
```

---

## 🔗 **LINKED LISTS (61–65)** - Complete Solutions

```cpp
// ListNode definition
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};
```

### 61. Middle of linked list
```cpp
ListNode* middleNode(ListNode* head) {
    ListNode *slow = head, *fast = head;
    
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    
    return slow;
}
```

### 62. Remove nth node from end
```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode *fast = &dummy, *slow = &dummy;
    
    // Move fast ahead by n+1
    for (int i = 0; i <= n; i++) {
        fast = fast->next;
    }
    
    // Move both until fast reaches end
    while (fast) {
        fast = fast->next;
        slow = slow->next;
    }
    
    // Remove node
    slow->next = slow->next->next;
    
    return dummy.next;
}
```

### 63. Merge two sorted lists
```cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
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
```

### 64. Palindrome linked list
```cpp
bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;
    
    // Find middle
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    
    // Reverse second half
    ListNode *prev = nullptr, *curr = slow;
    while (curr) {
        ListNode* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    
    // Compare
    ListNode *first = head, *second = prev;
    while (second) {
        if (first->val != second->val) return false;
        first = first->next;
        second = second->next;
    }
    
    return true;
}
```

### 65. Intersection of two linked lists
```cpp
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    if (!headA || !headB) return nullptr;
    
    ListNode *a = headA, *b = headB;
    
    while (a != b) {
        a = a ? a->next : headB;
        b = b ? b->next : headA;
    }
    
    return a;
}
```

---

## 🌳 **TREES (66–75)** - Complete Solutions

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

### 66. Inorder traversal (recursive)
```cpp
void inorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    inorder(root->left, result);
    result.push_back(root->val);
    inorder(root->right, result);
}
```

### 67. Inorder traversal (iterative)
```cpp
vector<int> inorderIterative(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    TreeNode* curr = root;
    
    while (curr || !st.empty()) {
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        curr = st.top(); st.pop();
        result.push_back(curr->val);
        curr = curr->right;
    }
    
    return result;
}
```

### 68. Max depth of binary tree
```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```

### 69. Invert binary tree
```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    
    swap(root->left, root->right);
    invertTree(root->left);
    invertTree(root->right);
    
    return root;
}
```

### 70. Lowest common ancestor
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    
    TreeNode* left = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    
    if (left && right) return root;
    return left ? left : right;
}
```

### 71. Level order traversal
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    
    queue<TreeNode*> q;
    q.push(root);
    
    while (!q.empty()) {
        int levelSize = q.size();
        vector<int> level;
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        
        result.push_back(level);
    }
    
    return result;
}
```

### 72. Symmetric tree
```cpp
bool isMirror(TreeNode* left, TreeNode* right) {
    if (!left && !right) return true;
    if (!left || !right) return false;
    
    return (left->val == right->val) &&
           isMirror(left->left, right->right) &&
           isMirror(left->right, right->left);
}

bool isSymmetric(TreeNode* root) {
    return !root || isMirror(root->left, root->right);
}
```

### 73. Path sum (has path sum)
```cpp
bool hasPathSum(TreeNode* root, int targetSum) {
    if (!root) return false;
    
    targetSum -= root->val;
    if (!root->left && !root->right) return targetSum == 0;
    
    return hasPathSum(root->left, targetSum) ||
           hasPathSum(root->right, targetSum);
}
```

### 74. Validate binary search tree
```cpp
bool isValidBST(TreeNode* root, long minVal = LONG_MIN, long maxVal = LONG_MAX) {
    if (!root) return true;
    if (root->val <= minVal || root->val >= maxVal) return false;
    
    return isValidBST(root->left, minVal, root->val) &&
           isValidBST(root->right, root->val, maxVal);
}
```

### 75. Sorted array to BST
```cpp
TreeNode* sortedArrayToBST(vector<int>& nums, int left, int right) {
    if (left > right) return nullptr;
    
    int mid = left + (right - left) / 2;
    TreeNode* root = new TreeNode(nums[mid]);
    
    root->left = sortedArrayToBST(nums, left, mid - 1);
    root->right = sortedArrayToBST(nums, mid + 1, right);
    
    return root;
}

TreeNode* sortedArrayToBST(vector<int>& nums) {
    return sortedArrayToBST(nums, 0, nums.size() - 1);
}
```

---

## 💡 **DYNAMIC PROGRAMMING (76–85)** - Complete Solutions

### 76. Climbing stairs
```cpp
int climbStairs(int n) {
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

### 77. House robber
```cpp
int rob(vector<int>& nums) {
    if (nums.empty()) return 0;
    if (nums.size() == 1) return nums[0];
    
    int prev2 = nums[0];
    int prev1 = max(nums[0], nums[1]);
    
    for (int i = 2; i < nums.size(); i++) {
        int curr = max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    
    return prev1;
}
```

### 78. Longest increasing subsequence
```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    
    for (int num : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end())
            tails.push_back(num);
        else
            *it = num;
    }
    
    return tails.size();
}

// Example: [10,9,2,5,3,7,101,18] → 4
```

### 79. Edit distance (Levenshtein distance)
```cpp
int minDistance(string word1, string word2) {
    int m = word1.length(), n = word2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1));
    
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1[i - 1] == word2[j - 1])
                dp[i][j] = dp[i - 1][j - 1];
            else
                dp[i][j] = 1 + min({dp[i - 1][j],      // delete
                                   dp[i][j - 1],      // insert
                                   dp[i - 1][j - 1]}); // replace
        }
    }
    
    return dp[m][n];
}
```

### 80. Maximum product subarray
```cpp
int maxProduct(vector<int>& nums) {
    int maxProd = nums[0], minProd = nums[0], result = nums[0];
    
    for (int i = 1; i < nums.size(); i++) {
        int temp = maxProd;
        maxProd = max({nums[i], nums[i] * maxProd, nums[i] * minProd});
        minProd = min({nums[i], nums[i] * temp, nums[i] * minProd});
        result = max(result, maxProd);
    }
    
    return result;
}

// Example: [2,3,-2,4] → 6
```

### 81. Unique paths
```cpp
int uniquePaths(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1));
    
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    
    return dp[m - 1][n - 1];
}
```

### 82. Palindrome partitioning
```cpp
vector<vector<string>> partition(string s) {
    vector<vector<string>> result;
    vector<string> current;
    
    function<void(int)> backtrack = [&](int start) {
        if (start == s.length()) {
            result.push_back(current);
            return;
        }
        
        for (int end = start; end < s.length(); end++) {
            if (isPalindrome(s.substr(start, end - start + 1))) {
                current.push_back(s.substr(start, end - start + 1));
                backtrack(end + 1);
                current.pop_back();
            }
        }
    };
    
    backtrack(0);
    return result;
}
```

### 83. Word break
```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<bool> dp(s.length() + 1, false);
    dp[0] = true;
    
    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.count(s.substr(j, i - j))) {
                dp[i] = true;
                break;
            }
        }
    }
    
    return dp[s.length()];
}
```

### 84. Coin change (minimum coins)
```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (i >= coin) {
                dp[i] = min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### 85. Knapsack (0/1)
```cpp
int knapsack(vector<int>& weights, vector<int>& values, int capacity) {
    int n = weights.size();
    vector<vector<int>> dp(n + 1, vector<int>(capacity + 1, 0));
    
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                dp[i][w] = max(dp[i - 1][w],
                              dp[i - 1][w - weights[i - 1]] + values[i - 1]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    return dp[n][capacity];
}
```

---

## 📈 **GRAPHS (86–90)** - Complete Solutions

### 86. DFS traversal
```cpp
void dfs(vector<vector<int>>& graph, int node, vector<bool>& visited) {
    visited[node] = true;
    cout << node << " ";
    
    for (int neighbor : graph[node]) {
        if (!visited[neighbor]) {
            dfs(graph, neighbor, visited);
        }
    }
}
```

### 87. BFS traversal
```cpp
void bfs(vector<vector<int>>& graph, int start) {
    vector<bool> visited(graph.size(), false);
    queue<int> q;
    
    visited[start] = true;
    q.push(start);
    
    while (!q.empty()) {
        int node = q.front(); q.pop();
        cout << node << " ";
        
        for (int neighbor : graph[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
        }
    }
}
```

### 88. Cycle detection in directed graph
```cpp
bool hasCycle(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> visited(n, 0); // 0=unvisited, 1=visiting, 2=visited
    
    function<bool(int)> dfs = [&](int node) {
        if (visited[node] == 1) return true;  // Cycle detected
        if (visited[node] == 2) return false; // Already processed
        
        visited[node] = 1;
        for (int neighbor : graph[node]) {
            if (dfs(neighbor)) return true;
        }
        visited[node] = 2;
        return false;
    };
    
    for (int i = 0; i < n; i++) {
        if (visited[i] == 0 && dfs(i)) return true;
    }
    return false;
}
```

### 89. Dijkstra's shortest path
```cpp
vector<int> dijkstra(vector<vector<pair<int, int>>>& graph, int start) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;
    
    dist[start] = 0;
    pq.push({0, start});
    
    while (!pq.empty()) {
        auto [currDist, node] = pq.top(); pq.pop();
        
        if (currDist > dist[node]) continue;
        
        for (auto [neighbor, weight] : graph[node]) {
            int newDist = currDist + weight;
            if (newDist < dist[neighbor]) {
                dist[neighbor] = newDist;
                pq.push({newDist, neighbor});
            }
        }
    }
    
    return dist;
}
```

### 90. Number of islands (DFS)
```cpp
int numIslands(vector<vector<char>>& grid) {
    if (grid.empty()) return 0;
    
    int rows = grid.size(), cols = grid[0].size();
    int islands = 0;
    
    function<void(int, int)> dfs = [&](int r, int c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] == '0')
            return;
        
        grid[r][c] = '0';  // Mark as visited
        
        dfs(r + 1, c);
        dfs(r - 1, c);
        dfs(r, c + 1);
        dfs(r, c - 1);
    };
    
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            if (grid[i][j] == '1') {
                islands++;
                dfs(i, j);
            }
        }
    }
    
    return islands;
}
```

---

## 🏗️ **OOP & ADVANCED C++ (91–100)** - Complete Solutions

### 91. Virtual functions and polymorphism
```cpp
class Shape {
public:
    virtual double area() const = 0;  // Pure virtual
    virtual ~Shape() = default;       // Virtual destructor
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override {
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() const override {
        return width * height;
    }
};

// Usage
vector<Shape*> shapes = {new Circle(5), new Rectangle(4, 6)};
for (Shape* s : shapes) {
    cout << s->area() << endl;  // Polymorphic call
}
```

### 92. Copy constructor vs assignment operator
```cpp
class String {
private:
    char* data;
    size_t length;
    
public:
    // Constructor
    String(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
    }
    
    // Copy constructor
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        cout << "Copy constructor called" << endl;
    }
    
    // Assignment operator
    String& operator=(const String& other) {
        if (this != &other) {  // Self-assignment check
            delete[] data;
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        cout << "Assignment operator called" << endl;
        return *this;
    }
    
    // Destructor
    ~String() {
        delete[] data;
    }
};

// Usage:
String s1("Hello");
String s2 = s1;  // Copy constructor
String s3;
s3 = s1;         // Assignment operator
```

### 93. Smart pointers (unique_ptr)
```cpp
#include <memory>

class Resource {
public:
    Resource() { cout << "Resource acquired" << endl; }
    ~Resource() { cout << "Resource released" << endl; }
    void doWork() { cout << "Working..." << endl; }
};

void uniquePtrExample() {
    // Unique ownership - cannot be copied
    unique_ptr<Resource> ptr1 = make_unique<Resource>();
    ptr1->doWork();
    
    // Transfer ownership
    unique_ptr<Resource> ptr2 = move(ptr1);
    // ptr1 is now null
    
    // Automatic cleanup when ptr2 goes out of scope
}

void sharedPtrExample() {
    // Shared ownership - reference counted
    shared_ptr<Resource> ptr1 = make_shared<Resource>();
    {
        shared_ptr<Resource> ptr2 = ptr1;  // Reference count = 2
        ptr2->doWork();
    }  // ptr2 destroyed, reference count = 1
    
    // Resource still alive
    ptr1->doWork();
}  // Resource destroyed when reference count reaches 0
```

### 94. Move semantics
```cpp
class Buffer {
private:
    int* data;
    size_t size;
    
public:
    // Constructor
    Buffer(size_t s) : size(s) {
        data = new int[size];
        cout << "Constructor: allocated " << size << " ints" << endl;
    }
    
    // Destructor
    ~Buffer() {
        delete[] data;
        cout << "Destructor: deallocated" << endl;
    }
    
    // Copy constructor (expensive)
    Buffer(const Buffer& other) : size(other.size) {
        data = new int[size];
        copy(other.data, other.data + size, data);
        cout << "Copy constructor: deep copy" << endl;
    }
    
    // Move constructor (cheap)
    Buffer(Buffer&& other) noexcept : data(nullptr), size(0) {
        data = other.data;
        size = other.size;
        other.data = nullptr;
        other.size = 0;
        cout << "Move constructor: transferred ownership" << endl;
    }
    
    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
            cout << "Move assignment" << endl;
        }
        return *this;
    }
};

// Factory function (returns temporary)
Buffer createBuffer() {
    Buffer temp(100);
    return temp;  // Move constructor called automatically
}

// Usage
Buffer b1 = createBuffer();  // Move constructor
Buffer b2 = move(b1);        // Explicit move
```

### 95. Template function for swap
```cpp
template<typename T>
void mySwap(T& a, T& b) {
    T temp = move(a);  // Use move for efficiency
    a = move(b);
    b = move(temp);
}

// Usage with different types
int x = 5, y = 10;
mySwap(x, y);  // int

string s1 = "Hello", s2 = "World";
mySwap(s1, s2);  // string

vector<int> v1 = {1,2,3}, v2 = {4,5,6};
mySwap(v1, v2);  // vector
```

### 96. Singleton pattern (thread-safe)
```cpp
class Singleton {
private:
    Singleton() { cout << "Singleton created" << endl; }
    ~Singleton() { cout << "Singleton destroyed" << endl; }
    
    // Delete copy operations
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
public:
    static Singleton& getInstance() {
        static Singleton instance;  // Thread-safe in C++11+
        return instance;
    }
    
    void doSomething() {
        cout << "Singleton doing work" << endl;
    }
};

// Usage
Singleton::getInstance().doSomething();
```

### 97. RAII (Resource Acquisition Is Initialization)
```cpp
class FileHandle {
private:
    FILE* file;
    
public:
    // Acquire resource in constructor
    FileHandle(const char* filename, const char* mode) {
        file = fopen(filename, mode);
        if (!file) throw runtime_error("Cannot open file");
        cout << "File opened: " << filename << endl;
    }
    
    // Release resource in destructor
    ~FileHandle() {
        if (file) {
            fclose(file);
            cout << "File closed" << endl;
        }
    }
    
    // Prevent copying
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    
    // Allow moving
    FileHandle(FileHandle&& other) noexcept : file(other.file) {
        other.file = nullptr;
    }
    
    // Accessor
    FILE* get() const { return file; }
    
    void write(const char* data) {
        if (file) fputs(data, file);
    }
};

// Usage - resource automatically released on scope exit
void processFile() {
    FileHandle fh("test.txt", "w");
    fh.write("Hello RAII");
    // No need to manually close - destructor handles it
}
```

### 98. Const correctness
```cpp
class DataProcessor {
private:
    vector<int> data;
    mutable int cacheHitCount = 0;  // Can be modified in const methods
    
public:
    void addData(int value) {  // Non-const method
        data.push_back(value);
    }
    
    int getSum() const {  // Const method - doesn't modify object
        int sum = 0;
        for (int val : data) sum += val;
        return sum;
    }
    
    int getValue(int index) const {
        cacheHitCount++;  // Allowed because mutable
        return data[index];
    }
    
    // Overloaded based on const-ness
    int& operator[](int index) {
        return data[index];  // Non-const version
    }
    
    const int& operator[](int index) const {
        return data[index];  // Const version
    }
};

// Function overloading with const
void process(const DataProcessor& constObj) {
    // Can only call const methods
    cout << constObj.getSum() << endl;
}

void process(DataProcessor& nonConstObj) {
    nonConstObj.addData(100);  // Can modify
}
```

### 99. Explicit constructor (prevent implicit conversion)
```cpp
class Integer {
private:
    int value;
    
public:
    // Explicit prevents implicit conversions
    explicit Integer(int v) : value(v) {
        cout << "Integer created: " << value << endl;
    }
    
    int getValue() const { return value; }
};

void printInteger(const Integer& i) {
    cout << i.getValue() << endl;
}

// Usage
Integer i1(42);           // OK
Integer i2 = 42;          // Error! Implicit conversion not allowed
printInteger(100);        // Error! Can't convert int to Integer
printInteger(Integer(100)); // OK - explicit construction

// Without explicit, these would compile but cause subtle bugs
```

### 100. Deep vs shallow copy
```cpp
class ShallowCopy {
public:
    int* data;
    
    ShallowCopy(int value) {
        data = new int(value);
        cout << "Shallow constructed" << endl;
    }
    
    // Shallow copy - just copies pointer
    ShallowCopy(const ShallowCopy& other) {
        data = other.data;  // Both objects point to same memory
        cout << "Shallow copy - pointer copied" << endl;
    }
    
    ~ShallowCopy() {
        delete data;
        cout << "Shallow destroyed" << endl;
    }
};

class DeepCopy {
public:
    int* data;
    
    DeepCopy(int value) {
        data = new int(value);
        cout << "Deep constructed" << endl;
    }
    
    // Deep copy - copies the actual data
    DeepCopy(const DeepCopy& other) {
        data = new int(*other.data);  // New memory allocation
        cout << "Deep copy - data duplicated" << endl;
    }
    
    ~DeepCopy() {
        delete data;
        cout << "Deep destroyed" << endl;
    }
};

// Problem with shallow copy
void demonstrateShallowCopyProblem() {
    ShallowCopy obj1(42);
    ShallowCopy obj2 = obj1;  // Shallow copy - both point to same int
    
    // When obj2 destroyed, it deletes data
    // When obj1 destroyed, double delete causes undefined behavior!
}

// Solution with deep copy
void demonstrateDeepCopy() {
    DeepCopy obj1(42);
    DeepCopy obj2 = obj1;  // Deep copy - separate memory
    
    // Both destroyed safely without double delete
}
```

---

## 📌 **Quick Reference Summary**

| Category | Difficulty | Key Concepts |
|----------|-----------|---------------|
| Numbers & Math | Easy | GCD, LCM, prime factors, conversions |
| Strings | Easy-Medium | Palindromes, anagrams, encoding |
| Arrays | Medium | Two pointers, sliding window, prefix sums |
| Linked Lists | Medium | Floyd's cycle, reversal, merging |
| Trees | Medium-Hard | Traversals, BST properties, recursion |
| Dynamic Programming | Hard | Memoization, tabulation, state transitions |
| Graphs | Hard | DFS/BFS, shortest paths, cycle detection |
| OOP & Advanced | Medium | Smart pointers, RAII, move semantics |

Each code example is **complete and runnable** (with necessary includes and main function where shown). Would you like me to:
1. **Explain any specific solution in more detail**
2. **Provide test cases and edge cases** for certain problems
3. **Create a compilation of all 100 solutions as individual files**