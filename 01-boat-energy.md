# Maximum Distance with Energy Constraint

## Problem Statement

You are given:
- Maximum initial energy `k` (k < 1000)
- An array `A` of length `n` denoting wind speed on `n` days
- A person is stuck on a boat

Each day, the person can choose to either:
- **Travel**: Move ahead by `A[i]` distance, energy decreases by 1
- **Stay/Rest**: Distance remains unchanged, energy increases by 1 (but cannot exceed `k`)

**Important Constraints:**
- Energy must never drop to negative
- Energy cannot exceed the maximum `k` (if energy is already `k` and person rests, energy stays at `k`)
- If `A[i]` is negative, traveling reduces total distance

**Goal:** Find the maximum distance the person can travel after `n` days without dropping energy to negative.

## Examples

### Example 1
```
Input: n = 3, k = 2, A = [5, 3, 4]
Output: 9
Explanation: 
- Day 0: Travel → energy = 1, distance = 5
- Day 1: Rest → energy = 2 (capped at k)
- Day 2: Travel → energy = 1, distance = 5 + 4 = 9
Total distance = 9
```

### Example 2
```
Input: n = 4, k = 3, A = [10, 5, 2, 8]
Output: 23
Explanation:
- Day 0: Travel → energy = 2, distance = 10
- Day 1: Travel → energy = 1, distance = 15
- Day 2: Rest → energy = 2
- Day 3: Travel → energy = 1, distance = 23
Total distance = 23
```

### Example 3
```
Input: n = 3, k = 2, A = [-5, -3, -4]
Output: 0
Explanation:
Since all A[i] are negative, traveling always decreases distance.
The optimal strategy is to never travel (always rest).
Total distance = 0, energy remains at k.
```

### Example 4
```
Input: n = 3, k = 1, A = [10, 5, 8]
Output: 18
Explanation:
- Day 0: Travel → energy = 0, distance = 10
- Day 1: Rest → energy = 1 (recovered from 0)
- Day 2: Travel → energy = 0, distance = 10 + 8 = 18
Total distance = 18
Note: We skip traveling on day 1 (A[1]=5) to save energy for day 2 (A[2]=8) which gives more distance.
```

## Constraints
- `1 <= n <= 1000`
- `1 <= k < 1000`
- `-10^9 <= A[i] <= 10^9` (A[i] can be negative)
- Energy must never be negative
- Energy cannot exceed `k`
- If energy is already `k` and person rests, energy stays at `k`

## Approach

### Step 1: Understanding the Problem
We need to maximize distance traveled over `n` days with energy constraints. At each day, we have two choices:
- Travel (if energy > 0): Gain distance `A[i]`, lose 1 energy
- Rest: No distance change, gain 1 energy (capped at `k`)

This is an optimization problem with constraints, making it suitable for Dynamic Programming.

### Step 2: Identifying the Strategy
This problem exhibits:
- **Optimal Substructure**: The maximum distance from day `i` depends on optimal choices from day `i+1`
- **Overlapping Subproblems**: Same (day, energy) states are reached multiple times
- **Decision Tree**: At each day, we choose between travel and rest

We'll use **Recursion with Memoization (Top-down DP)** to explore all valid paths and find the maximum distance.

### Step 3: Algorithm Design
1. **State Definition**: `solve(day, energy)` returns maximum distance possible from `day` to `n-1` with current `energy`
2. **Base Case**: If `day == n`, return 0 (no more days left)
3. **Transitions**:
   - **Rest**: `solve(day + 1, min(k, energy + 1))` - Energy increases by 1, capped at k
   - **Travel**: `A[day] + solve(day + 1, energy - 1)` - Only if `energy > 0` and preferably `A[day] > 0`
4. **Memoization**: Store results in `dp[day][energy]` to avoid recomputation
5. **Return**: Maximum of rest and travel options

### Step 4: Edge Cases
- Energy already at maximum `k`: Resting keeps energy at `k` (use `min(k, energy + 1)`)
- Energy at 0: Cannot travel (must rest)
- All `A[i] < 0`: Optimal to never travel (answer is 0)
- Negative `A[i]`: Should avoid traveling on negative days (though DP will naturally reject it via `max()`)

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
private:
    int n, k;
    vector<int> A;
    vector<vector<int>> dp;
    
    int solve(int day, int energy) {
        // Base case: no more days left
        if (day == n)
            return 0;
        
        // If already computed, return memoized result
        if (dp[day][energy] != -1)
            return dp[day][energy];
        
        // Option 1: Rest
        // Energy increases by 1, but capped at k
        int rest = solve(day + 1, min(k, energy + 1));
        
        // Option 2: Travel (only if energy > 0)
        int travel = INT_MIN;
        if (energy > 0) {
            travel = A[day] + solve(day + 1, energy - 1);
        }
        
        // Return maximum of both options
        // If travel is not possible (energy == 0), travel will be INT_MIN
        return dp[day][energy] = max(rest, travel);
    }
    
public:
    /**
     * Find maximum distance that can be traveled with energy constraints
     * 
     * @param n Number of days
     * @param k Maximum initial energy
     * @param A Array of wind speeds (distances) for each day
     * @return Maximum distance possible
     */
    int maxDistance(int n, int k, vector<int>& A) {
        this->n = n;
        this->k = k;
        this->A = A;
        
        // Initialize DP table: dp[day][energy]
        // Energy ranges from 0 to k
        dp.assign(n, vector<int>(k + 1, -1));
        
        // Start from day 0 with initial energy k
        return solve(0, k);
    }
};
```

### Explanation
- **Line 1-4**: Include necessary headers (`vector`, `algorithm` for `max`, `climits` for `INT_MIN`)
- **Line 6-10**: Class definition with private members: `n` (days), `k` (max energy), `A` (wind speeds), `dp` (memoization table)
- **Line 12-13**: Base case - when all days are processed, no more distance can be gained
- **Line 15-17**: Memoization check - if state `(day, energy)` is already computed, return stored result
- **Line 19-21**: **Rest option** - Energy increases by 1 but is capped at `k` using `min(k, energy + 1)`. This handles the edge case where energy is already at maximum.
- **Line 23-26**: **Travel option** - Only allowed if `energy > 0`. Distance increases by `A[day]` and energy decreases by 1.
- **Line 28-30**: Return the maximum of rest and travel. If travel is not possible, `travel` remains `INT_MIN`, so `max()` will choose rest.
- **Line 40-42**: Initialize DP table with dimensions `n × (k+1)`, all values set to `-1` (uncomputed)
- **Line 44**: Start recursion from day 0 with initial energy `k`

**Key Insight**: The `min(k, energy + 1)` directly in the function call elegantly handles the energy cap without needing a separate if-else block.

## Complexity Analysis

### Time Complexity
- **O(n × k)**: We have `n` days and energy can range from `0` to `k`, giving us `n × (k+1)` unique states
- Each state is computed exactly once due to memoization
- Each computation involves constant time operations (max, min, addition)

### Space Complexity
- **O(n × k)**: The DP memoization table requires `n × (k+1)` space
- **O(n)**: Recursion stack depth can be at most `n` (one level per day)
- **Overall**: O(n × k) dominated by the DP table

## Key Insights
- **Energy Cap Handling**: Using `min(k, energy + 1)` directly in the recursive call is cleaner than explicit if-else
- **Negative A[i] Optimization**: If all `A[i] < 0`, the optimal answer is 0 (never travel). The DP naturally handles this as `max(rest, travel)` will always choose rest when travel gives negative distance.
- **State Space**: The problem has `n × k` states, making it efficiently solvable with DP
- **Greedy Fails**: A greedy approach (always travel when energy > 0 and A[i] > 0) fails because we might need to save energy for better future opportunities
- **Optimal Substructure**: The maximum distance from any state depends only on optimal choices from future states

## Alternative Approaches

### Bottom-Up DP (Iterative)
```cpp
int maxDistance(int n, int k, vector<int>& A) {
    vector<vector<int>> dp(n + 1, vector<int>(k + 1, 0));
    
    // Start from last day and work backwards
    for (int day = n - 1; day >= 0; day--) {
        for (int energy = 0; energy <= k; energy++) {
            // Rest option
            int rest = dp[day + 1][min(k, energy + 1)];
            
            // Travel option
            int travel = INT_MIN;
            if (energy > 0) {
                travel = A[day] + dp[day + 1][energy - 1];
            }
            
            dp[day][energy] = max(rest, travel);
        }
    }
    
    return dp[0][k];
}
```

**Advantages:**
- No recursion stack overhead
- Slightly better cache locality
- Can be space-optimized to O(k) if we only need final answer

**Time Complexity:** O(n × k)  
**Space Complexity:** O(n × k), can be optimized to O(k)

### Greedy Approach (Why It Fails)
A greedy strategy of "always travel when energy > 0 and A[i] > 0" fails because:
- Example: `k = 1, A = [1, 100]`
- Greedy: Travel on day 0 → energy = 0, distance = 1
- Optimal: Rest on day 0 → energy = 1, travel on day 1 → distance = 100
- Greedy gives 1, optimal gives 100

## Related Problems
- Maximum Subarray Sum with Constraints
- Knapsack Problem (0/1 Knapsack)
- Coin Change Problem
- House Robber with Energy Constraints
- Maximum Points You Can Obtain from Cards

