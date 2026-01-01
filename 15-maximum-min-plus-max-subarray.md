# Maximum Min Plus Max Subarray

## Problem Statement

Given an array of **positive numbers**, find a **subarray of size > 1** with the **maximum value of (minimum + maximum)** in that subarray.

In other words, for a subarray, calculate `min(subarray) + max(subarray)`, and find the subarray where this sum is maximum.

**Input Format:**
- `arr`: Vector of positive integers

**Output:** Maximum value of (min + max) over all subarrays of size > 1.

## Examples

### Example 1
```
Input: arr = [3, 5, 7, 1]

All subarrays of size > 1:
- [3, 5]: min=3, max=5, sum=8
- [3, 5, 7]: min=3, max=7, sum=10
- [3, 5, 7, 1]: min=1, max=7, sum=8
- [5, 7]: min=5, max=7, sum=12
- [5, 7, 1]: min=1, max=7, sum=8
- [7, 1]: min=1, max=7, sum=8

Maximum: 12 (from subarray [5, 7])
Output: 12
```

### Example 2
```
Input: arr = [10, 2, 8, 15, 3]

Key subarrays:
- [10, 2]: min=2, max=10, sum=12
- [8, 15]: min=8, max=15, sum=23
- [2, 8, 15]: min=2, max=15, sum=17
- [8, 15, 3]: min=3, max=15, sum=18

Maximum: 23 (from subarray [8, 15])
Output: 23
```

### Example 3
```
Input: arr = [1, 2, 3, 4, 5]

All adjacent pairs:
- [1, 2]: sum=3
- [2, 3]: sum=5
- [3, 4]: sum=7
- [4, 5]: sum=9

Maximum: 9 (from subarray [4, 5])
Output: 9
```

## Constraints
- `2 <= arr.length <= 10^5`
- `1 <= arr[i] <= 10^9`
- All numbers are positive

## Approach

### Step 1: Understanding the Problem
We need to find a subarray (contiguous elements) of size > 1 that maximizes `min(subarray) + max(subarray)`.

**Key Insight:** To maximize (min + max):
- One element should be the **maximum possible** (global max or near it)
- The other element should be as **large as possible** but can be smaller than the maximum
- This situation occurs naturally in **subarrays of size 2** (adjacent pairs)

### Step 2: Identifying the Strategy
**Why Size 2 is Optimal:**
- In a subarray of size 2: `min + max = a + b` (sum of two elements)
- In a subarray of size > 2: `min + max ≤ max + second_max` (min is at most second_max)
- The maximum sum occurs when we have the **largest element paired with the second largest element**
- This is best achieved in **adjacent pairs** of the largest elements

**Greedy Approach:**
- Check all **adjacent pairs** (subarrays of size 2)
- The maximum (min + max) will be found among these pairs
- No need to check larger subarrays

### Step 3: Algorithm Design
1. **Edge case**: If array size < 2, return 0
2. **Check all adjacent pairs**: For `i` from 0 to `n-2`:
   - Calculate `arr[i] + arr[i+1]`
   - Update maximum
3. **Return** maximum sum found

### Step 4: Edge Cases
- Array size < 2: Return 0 (no valid subarray)
- All elements same: Return `2 × element` (any pair works)
- Two elements: Return their sum

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    /**
     * Find maximum (min + max) value in any subarray of size > 1
     * 
     * @param arr Vector of positive integers
     * @return Maximum value of (min + max) over all subarrays
     */
    int maxMinPlusMaxSubarray(vector<int>& arr) {
        int n = arr.size();
        
        // Edge case: need at least 2 elements for valid subarray
        if (n < 2) {
            return 0;
        }
        
        int ans = 0;
        
        // Only check subarrays of size 2 (adjacent pairs)
        // Key insight: Maximum (min + max) occurs in pairs of largest elements
        for (int i = 0; i < n - 1; i++) {
            // For subarray [arr[i], arr[i+1]]:
            // min = min(arr[i], arr[i+1])
            // max = max(arr[i], arr[i+1])
            // min + max = arr[i] + arr[i+1] (sum of the two elements)
            ans = max(ans, arr[i] + arr[i+1]);
        }
        
        return ans;
    }
};
```

### Explanation
- **Line 14-17**: Edge case handling: return 0 if array has less than 2 elements
- **Line 19**: Initialize answer to 0
- **Line 21-28**: Check all adjacent pairs:
  - **Line 22**: Iterate through all adjacent pairs `(i, i+1)`
  - **Line 23-26**: For a subarray of size 2:
    - `min = min(arr[i], arr[i+1])`
    - `max = max(arr[i], arr[i+1])`
    - `min + max = arr[i] + arr[i+1]` (always the sum of both elements)
  - **Line 27**: Update maximum
- **Line 30**: Return maximum sum found

**Why This Works:**
- For any subarray of size 2: `min + max = sum of both elements`
- For larger subarrays: `min + max ≤ max + second_max ≤ sum of two largest adjacent elements`
- The maximum occurs when we pair the two largest possible adjacent elements
- Checking all adjacent pairs guarantees we find the maximum

**Mathematical Proof:**
- For subarray `[a, b]`: `min + max = a + b`
- For subarray `[a, b, c]`: `min + max ≤ max(a,b,c) + second_max(a,b,c) ≤ a + b` or `b + c` or `a + c`
- The maximum of all adjacent pairs will be at least as large as any larger subarray

## Complexity Analysis

### Time Complexity
- **O(n)**: 
  - Single pass through array
  - Check `n-1` adjacent pairs
  - Each operation is O(1)
  - **Overall**: O(n)

### Space Complexity
- **O(1)**: 
  - Only using a few variables
  - No extra data structures
  - **Overall**: O(1)

## Key Insights
- **Size 2 is Optimal**: Maximum (min + max) always occurs in subarrays of size 2
- **Adjacent Pairs**: We only need to check adjacent pairs, not all possible subarrays
- **Simple Sum**: For size 2, min + max = sum of both elements
- **Greedy Approach**: No need for complex algorithms - just find maximum sum of adjacent pairs
- **Why Not Larger Subarrays**: Adding more elements can only decrease the minimum, not increase the maximum

## Alternative Approaches

### Brute Force (Check All Subarrays)

```cpp
int maxMinPlusMaxSubarrayBruteForce(vector<int>& arr) {
    int n = arr.size();
    if (n < 2) return 0;
    
    int ans = 0;
    
    // Check all subarrays of size >= 2
    for (int i = 0; i < n; i++) {
        int minVal = arr[i];
        int maxVal = arr[i];
        
        for (int j = i + 1; j < n; j++) {
            minVal = min(minVal, arr[j]);
            maxVal = max(maxVal, arr[j]);
            ans = max(ans, minVal + maxVal);
        }
    }
    
    return ans;
}
```

**Time Complexity:** O(n²)  
**Space Complexity:** O(1)

**Why Not Needed:**
- The optimal solution only needs O(n)
- Brute force is unnecessary

### Using Sliding Window (Unnecessary)

```cpp
int maxMinPlusMaxSubarraySlidingWindow(vector<int>& arr) {
    int n = arr.size();
    if (n < 2) return 0;
    
    int ans = 0;
    
    // Sliding window of size 2
    for (int i = 0; i <= n - 2; i++) {
        int minVal = min(arr[i], arr[i+1]);
        int maxVal = max(arr[i], arr[i+1]);
        ans = max(ans, minVal + maxVal);
    }
    
    return ans;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(1)

**Note:** This is essentially the same as the optimal solution, just more verbose.

## Related Problems
- Maximum Sum Subarray
- Maximum Product Subarray
- Subarray Sum Equals K
- Maximum Average Subarray

