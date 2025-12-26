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

```python
def twoSum(nums, target):
    """
    Find two indices whose values sum to target.
    
    Args:
        nums: List of integers
        target: Target sum
    
    Returns:
        List of two indices [i, j] where nums[i] + nums[j] == target
    """
    # Hash map to store value -> index mapping
    num_map = {}
    
    # Iterate through the array
    for i, num in enumerate(nums):
        # Calculate the complement
        complement = target - num
        
        # Check if complement exists in the map
        if complement in num_map:
            # Found the pair!
            return [num_map[complement], i]
        
        # Store current number and its index
        num_map[num] = i
    
    # No solution found (shouldn't happen per problem constraints)
    return []
```

### Explanation
- **Line 8-9**: Create a hash map to store numbers we've seen and their indices
- **Line 11-12**: Iterate through the array with both index and value
- **Line 14**: Calculate what number we need to complete the sum
- **Line 16-19**: If we've seen the complement before, return both indices
- **Line 21-22**: Otherwise, store the current number for future lookups

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
```python
def twoSumBruteForce(nums, target):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []
```

### Two-Pass Hash Map (O(n))
```python
def twoSumTwoPass(nums, target):
    num_map = {}
    # First pass: build the map
    for i, num in enumerate(nums):
        num_map[num] = i
    
    # Second pass: find the complement
    for i, num in enumerate(nums):
        complement = target - num
        if complement in num_map and num_map[complement] != i:
            return [i, num_map[complement]]
    return []
```

## Related Problems
- Three Sum
- Four Sum
- Two Sum II - Input array is sorted
- Subarray Sum Equals K

