# Two Sum

## Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

You can return the answer in any order.

## Examples

### Example 1
```
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
```

### Example 2
```
Input: nums = [3,2,4], target = 6
Output: [1,2]
Explanation: Because nums[1] + nums[2] == 6, we return [1, 2].
```

### Example 3
```
Input: nums = [3,3], target = 6
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 6, we return [0, 1].
```

## Constraints
- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- Only one valid answer exists.

## Approach

### Step 1: Understanding the Problem
We need to find two indices in the array such that the values at those indices sum to the target. The naive approach would be to check all pairs, but we can optimize using a hash map.

### Step 2: Identifying the Strategy
Instead of checking all pairs (O(n²)), we can use a hash map to store each number and its index as we iterate. For each number, we check if `target - current_number` exists in the map.

### Step 3: Algorithm Design
1. Create an empty hash map to store `{value: index}` pairs
2. Iterate through the array with index `i`
3. For each element `nums[i]`, calculate `complement = target - nums[i]`
4. If `complement` exists in the hash map, return `[map[complement], i]`
5. Otherwise, add `{nums[i]: i}` to the hash map
6. Continue until a solution is found

### Step 4: Edge Cases
- Array with exactly 2 elements
- Negative numbers (handled by hash map)
- Duplicate numbers (last occurrence will overwrite, but we check before adding)

## Solution

### Code Implementation

```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    /**
     * Find two indices whose values sum to target.
     * 
     * @param nums Vector of integers
     * @param target Target sum
     * @return Vector of two indices [i, j] where nums[i] + nums[j] == target
     */
    vector<int> twoSum(vector<int>& nums, int target) {
        // Hash map to store value -> index mapping
        unordered_map<int, int> numMap;
        
        // Iterate through the array
        for (int i = 0; i < nums.size(); i++) {
            // Calculate the complement
            int complement = target - nums[i];
            
            // Check if complement exists in the map
            if (numMap.find(complement) != numMap.end()) {
                // Found the pair!
                return {numMap[complement], i};
            }
            
            // Store current number and its index
            numMap[nums[i]] = i;
        }
        
        // No solution found (shouldn't happen per problem constraints)
        return {};
    }
};
```

### Explanation
- **Line 1-3**: Include necessary headers (`vector` for arrays, `unordered_map` for hash map)
- **Line 5-6**: Define a Solution class (common in LeetCode-style problems)
- **Line 12**: Create an `unordered_map<int, int>` to store value -> index pairs
- **Line 14**: Iterate through the array using traditional for loop with index
- **Line 16**: Calculate the complement (target - current number)
- **Line 18-21**: Check if complement exists using `find()` method. If found, return both indices
- **Line 23**: Store the current number and its index in the map for future lookups


## Complexity Analysis

### Time Complexity
- **O(n)**: We iterate through the array once, and hash map operations (insertion and lookup) are O(1) on average

### Space Complexity
- **O(n)**: In the worst case, we might store all n elements in the hash map before finding the solution

## Key Insights
- Hash maps enable O(1) lookups, making this problem solvable in O(n) instead of O(n²)
- We can solve it in a single pass by storing seen values as we iterate
- The complement approach (target - current) is a common pattern in array problems

## Alternative Approaches

### Brute Force (O(n²))
```cpp
vector<int> twoSumBruteForce(vector<int>& nums, int target) {
    for (int i = 0; i < nums.size(); i++) {
        for (int j = i + 1; j < nums.size(); j++) {
            if (nums[i] + nums[j] == target) {
                return {i, j};
            }
        }
    }
    return {};
}
```

### Two-Pass Hash Map (O(n))
```cpp
vector<int> twoSumTwoPass(vector<int>& nums, int target) {
    unordered_map<int, int> numMap;
    
    // First pass: build the map
    for (int i = 0; i < nums.size(); i++) {
        numMap[nums[i]] = i;
    }
    
    // Second pass: find the complement
    for (int i = 0; i < nums.size(); i++) {
        int complement = target - nums[i];
        if (numMap.find(complement) != numMap.end() && numMap[complement] != i) {
            return {i, numMap[complement]};
        }
    }
    return {};
}
```

## Related Problems
- Three Sum
- Four Sum
- Two Sum II - Input array is sorted
- Subarray Sum Equals K

