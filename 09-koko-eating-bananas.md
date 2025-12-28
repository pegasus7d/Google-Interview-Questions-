# Koko Eating Bananas

## Problem Statement

Koko loves to eat bananas. There are `n` piles of bananas, where the `i`-th pile has `piles[i]` bananas. The guards will come back in `h` hours.

Koko can decide her **bananas-per-hour eating speed** of `k`. Each hour, she:
- Chooses some pile of bananas
- Eats `k` bananas from that pile
- If the pile has less than `k` bananas, she eats all of them and won't eat any more bananas during that hour

Koko wants to eat slowly but still finish eating all the bananas before the guards return.

Return the **minimum integer `k`** such that she can eat all the bananas within `h` hours.

## Examples

### Example 1
```
Input: piles = [3, 6, 7, 11], h = 8
Output: 4

Explanation:
If k = 4:
- Pile 0 (3): ⌈3/4⌉ = 1 hour
- Pile 1 (6): ⌈6/4⌉ = 2 hours
- Pile 2 (7): ⌈7/4⌉ = 2 hours
- Pile 3 (11): ⌈11/4⌉ = 3 hours
Total: 1 + 2 + 2 + 3 = 8 hours ✓

If k = 3:
- Pile 0 (3): ⌈3/3⌉ = 1 hour
- Pile 1 (6): ⌈6/3⌉ = 2 hours
- Pile 2 (7): ⌈7/3⌉ = 3 hours
- Pile 3 (11): ⌈11/3⌉ = 4 hours
Total: 1 + 2 + 3 + 4 = 10 hours ✗ (exceeds h=8)
```

### Example 2
```
Input: piles = [30, 11, 23, 4, 20], h = 5
Output: 30

Explanation:
If k = 30:
- Pile 0 (30): ⌈30/30⌉ = 1 hour
- Pile 1 (11): ⌈11/30⌉ = 1 hour
- Pile 2 (23): ⌈23/30⌉ = 1 hour
- Pile 3 (4): ⌈4/30⌉ = 1 hour
- Pile 4 (20): ⌈20/30⌉ = 1 hour
Total: 5 hours ✓

If k = 29:
- Total would be more than 5 hours ✗
```

### Example 3
```
Input: piles = [30, 11, 23, 4, 20], h = 6
Output: 23

Explanation:
If k = 23:
- Pile 0 (30): ⌈30/23⌉ = 2 hours
- Pile 1 (11): ⌈11/23⌉ = 1 hour
- Pile 2 (23): ⌈23/23⌉ = 1 hour
- Pile 3 (4): ⌈4/23⌉ = 1 hour
- Pile 4 (20): ⌈20/23⌉ = 1 hour
Total: 2 + 1 + 1 + 1 + 1 = 6 hours ✓
```

## Constraints
- `1 <= piles.length <= 10^4`
- `piles.length <= h <= 10^9`
- `1 <= piles[i] <= 10^9`

## Approach

### Step 1: Understanding the Problem
We need to find the minimum eating speed `k` such that:
- Koko can finish all piles within `h` hours
- For each pile with `p` bananas, it takes `⌈p/k⌉` hours to finish
- Total time = sum of `⌈piles[i]/k⌉` for all piles ≤ `h`

**Key Insight:** This is a **binary search on answer** problem. We're searching for the minimum `k` that satisfies the constraint.

### Step 2: Identifying the Strategy
**Why Binary Search?**
- If `k` works, then any `k' > k` also works (faster eating = less time)
- If `k` doesn't work, then any `k' < k` also doesn't work (slower eating = more time)
- This **monotonic property** makes binary search applicable

**Search Space:**
- **Lower bound**: `k = 1` (slowest possible)
- **Upper bound**: `k = max(piles)` (fastest needed - can finish any pile in 1 hour)

### Step 3: Algorithm Design
1. **Binary Search Setup:**
   - `low = 1`, `high = max(piles)`
   - Search for minimum `k` that works

2. **Check Function (`canFinish`):**
   - For given `k`, calculate total hours needed
   - For each pile: `hours += ⌈piles[i]/k⌉`
   - Use ceiling division: `(piles[i] + k - 1) / k`
   - Return `true` if total hours ≤ `h`, else `false`

3. **Binary Search Logic:**
   - If `canFinish(mid)` is true: `k = mid` works, try smaller (move `high = mid - 1`)
   - If `canFinish(mid)` is false: `k = mid` doesn't work, need larger (move `low = mid + 1`)

### Step 4: Edge Cases
- Single pile: Answer is `⌈piles[0]/h⌉`
- All piles same size: Answer is `⌈piles[0]/⌈h/n⌉⌉`
- Very large piles: Use `long long` to avoid overflow
- `h` equals number of piles: Must eat at least `max(piles)` per hour

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
public:
    /**
     * Check if Koko can finish all piles within h hours with speed k
     * 
     * @param piles Array of banana piles
     * @param h Available hours
     * @param k Eating speed (bananas per hour)
     * @return true if can finish, false otherwise
     */
    bool canFinish(vector<int>& piles, long long h, long long k) {
        long long hours = 0;
        
        for (int bananas : piles) {
            // Ceiling division: ⌈bananas/k⌉ = (bananas + k - 1) / k
            hours += (bananas + k - 1) / k;
            
            // Early termination if already exceeds h
            if (hours > h) {
                return false;
            }
        }
        
        return hours <= h;
    }
    
    /**
     * Find minimum eating speed to finish all bananas within h hours
     * 
     * @param piles Array of banana piles
     * @param h Available hours
     * @return Minimum eating speed k
     */
    int minEatingSpeed(vector<int>& piles, int h) {
        // Binary search bounds
        long long low = 1;
        long long high = *max_element(piles.begin(), piles.end());
        long long answer = high;  // Worst case: need to eat max(piles) per hour
        
        while (low <= high) {
            long long mid = low + (high - low) / 2;
            
            if (canFinish(piles, h, mid)) {
                // k = mid works, try to find smaller k
                answer = mid;
                high = mid - 1;
            } else {
                // k = mid doesn't work, need larger k
                low = mid + 1;
            }
        }
        
        return answer;
    }
};
```

### Explanation
- **Line 18-30**: `canFinish` function:
  - **Line 20**: Initialize total hours to 0
  - **Line 22-26**: For each pile:
    - **Line 24**: Calculate hours needed using ceiling division: `(bananas + k - 1) / k`
      - This is equivalent to `⌈bananas/k⌉`
      - Example: `⌈11/4⌉ = (11 + 4 - 1) / 4 = 14/4 = 3`
    - **Line 26-28**: Early termination if hours already exceed `h`
  - **Line 29**: Return true if total hours ≤ `h`
  
- **Line 37-58**: `minEatingSpeed` function:
  - **Line 40-42**: Set binary search bounds:
    - `low = 1` (minimum possible speed)
    - `high = max(piles)` (maximum needed - can finish any pile in 1 hour)
  - **Line 43**: Initialize answer to `high` (worst case)
  - **Line 45-56**: Binary search:
    - **Line 46**: Calculate mid point
    - **Line 48-52**: If `mid` works:
      - Update answer
      - Try smaller values (`high = mid - 1`)
    - **Line 53-55**: If `mid` doesn't work:
      - Need larger values (`low = mid + 1`)
  - **Line 58**: Return minimum valid `k`

**Why Ceiling Division?**
- If a pile has 11 bananas and `k = 4`:
  - Hour 1: Eat 4, remaining = 7
  - Hour 2: Eat 4, remaining = 3
  - Hour 3: Eat 3, done
  - Total: 3 hours = `⌈11/4⌉ = 3`

**Formula:** `⌈a/b⌉ = (a + b - 1) / b` for positive integers

## Complexity Analysis

### Time Complexity
- **O(n × log(max(piles)))**:
  - Binary search: O(log(max(piles))) iterations
  - Each iteration: O(n) to check all piles
  - **Overall**: O(n × log(max(piles)))

### Space Complexity
- **O(1)**: Only using a few variables, no extra data structures
- **Overall**: O(1)

## Key Insights
- **Binary Search on Answer**: When we're searching for a minimum/maximum value that satisfies a condition, and the condition is monotonic
- **Monotonic Property**: If `k` works, all larger `k` work; if `k` doesn't work, all smaller `k` don't work
- **Ceiling Division**: Use `(a + b - 1) / b` to compute `⌈a/b⌉` efficiently
- **Early Termination**: In `canFinish`, stop early if hours exceed `h` to save computation
- **Search Space**: Lower bound is 1, upper bound is `max(piles)` (can finish any pile in 1 hour)
- **Long Long**: Use `long long` to avoid integer overflow with large values

## Alternative Approaches

### Linear Search (Brute Force)

```cpp
int minEatingSpeedBruteForce(vector<int>& piles, int h) {
    int maxPile = *max_element(piles.begin(), piles.end());
    
    for (int k = 1; k <= maxPile; k++) {
        long long hours = 0;
        for (int bananas : piles) {
            hours += (bananas + k - 1) / k;
        }
        if (hours <= h) {
            return k;
        }
    }
    
    return maxPile;
}
```

**Time Complexity:** O(n × max(piles)) - Too slow for large values  
**Space Complexity:** O(1)

### Optimized Binary Search with Better Bounds

```cpp
int minEatingSpeedOptimized(vector<int>& piles, int h) {
    long long total = 0;
    int maxPile = 0;
    for (int p : piles) {
        total += p;
        maxPile = max(maxPile, p);
    }
    
    // Better lower bound: must eat at least ⌈total/h⌉ per hour
    long long low = (total + h - 1) / h;
    long long high = maxPile;
    long long answer = high;
    
    while (low <= high) {
        long long mid = low + (high - low) / 2;
        
        long long hours = 0;
        for (int bananas : piles) {
            hours += (bananas + mid - 1) / mid;
        }
        
        if (hours <= h) {
            answer = mid;
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    
    return answer;
}
```

**Time Complexity:** O(n × log(max(piles)))  
**Space Complexity:** O(1)

**Advantage:** Better lower bound reduces search space

## Related Problems
- Capacity To Ship Packages Within D Days
- Split Array Largest Sum
- Minimize Maximum Distance to Gas Station
- Divide Chocolate
- Minimize Maximum of Array


