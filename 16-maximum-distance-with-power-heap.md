# Maximum Distance with Power Constraint (Greedy Approach)

## Problem Statement

A sailor is on a boat and has **K units of power** initially. There is an array `arr` representing wind speeds on each day. Each day, the sailor can make one of two choices:

1. **Sail**: 
   - Power decreases by 1
   - Distance covered = `arr[i]` (wind speed on that day)

2. **Rest**:
   - Power increases by 1
   - Distance covered = 0

**Goal:** Find the **maximum total distance** the sailor can travel.

**Constraints:**
- Power can never go negative
- Power can increase beyond K (no upper limit)
- Sailor must process days in order (left to right)

**Input Format:**
- `arr`: Vector of wind speeds for each day
- `K`: Initial power units

**Output:** Maximum total distance that can be traveled.

## Examples

### Example 1
```
Input: arr = [5, 3, 7, 2], K = 2

Day 0 (wind=5): Power=2, Sail → Distance=5, Power=1
Day 1 (wind=3): Power=1, Sail → Distance=3, Power=0
Day 2 (wind=7): Power=0, Replace Day 1 (3 < 7) → Distance=7, Power=0
Day 3 (wind=2): Power=0, Not better than Day 0 (5) → Rest, Power=1

Total Distance: 5 + 7 = 12
Output: 12
```

### Example 2
```
Input: arr = [10, 8, 6, 4], K = 1

Day 0 (wind=10): Power=1, Sail → Distance=10, Power=0
Day 1 (wind=8): Power=0, Replace Day 0? No (10 > 8) → Rest, Power=1
Day 2 (wind=6): Power=1, Sail → Distance=6, Power=0
Day 3 (wind=4): Power=0, Replace Day 2? No (6 > 4) → Rest, Power=1

Total Distance: 10 + 6 = 16
Output: 16
```

### Example 3
```
Input: arr = [3, 1, 4, 1, 5], K = 1

Day 0 (wind=3): Power=1, Sail → Distance=3, Power=0
Day 1 (wind=1): Power=0, Replace Day 0? No (3 > 1) → Rest, Power=1
Day 2 (wind=4): Power=1, Sail → Distance=4, Power=0
Day 3 (wind=1): Power=0, Replace Day 2? No (4 > 1) → Rest, Power=1
Day 4 (wind=5): Power=1, Sail → Distance=5, Power=0

Total Distance: 3 + 4 + 5 = 12
Output: 12
```

## Constraints
- `1 <= arr.length <= 10^5`
- `1 <= arr[i] <= 10^9`
- `1 <= K <= 10^5`
- Power can increase beyond K (no upper limit)

## Approach

### Step 1: Understanding the Problem
Each day, the sailor has two choices:
- **Sail**: Spend 1 power, gain `arr[i]` distance
- **Rest**: Gain 1 power, gain 0 distance

**Key Constraint:** Power is limited and regenerates slowly, so we can't sail every day.

**Reframing:** Power behaves like a limited budget, and sailing on a day gives a profit equal to the wind speed. Since every sail costs exactly one power, we should spend power only on days with the highest wind speeds.

### Step 2: Identifying the Strategy
**Greedy Intuition:**
- We want to sail on days with the highest wind speeds
- The difficulty: days come in order, and we may see a better day after spending power
- Solution: Keep track of sailing days and be able to replace the worst one if a better day appears

**Data Structure Choice:**
- Use a **min-heap** to store wind speeds of days we decided to sail
- The smallest element represents the "worst" sailing day so far
- Allows quick identification and removal of the weakest sailing day in O(log n) time

### Step 3: Algorithm Design
1. **Initialize**: Min-heap, total distance = 0, power = K
2. **For each day**:
   - **Case 1: Have power** → Sail today, add wind to heap
   - **Case 2: No power** → Check if today's wind > worst sailing day:
     - If yes: Replace worst day with today
     - If no: Rest and regain power
3. **Return** total distance

**Why This Works:**
- At any point, the heap contains the best possible set of days to spend limited power on
- We always maintain the K (or fewer) best sailing days seen so far
- Total distance is always maximized

### Step 4: Edge Cases
- All wind speeds same: Any K days work
- Wind speeds in decreasing order: Sail first K days
- Wind speeds in increasing order: Replace previous days as better ones appear
- K = 0: Can only sail by replacing previous days

## Solution

### Code Implementation

```cpp
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

class Solution {
public:
    /**
     * Find maximum distance using greedy approach with min-heap
     * 
     * @param arr Wind speeds for each day
     * @param K Initial power units
     * @return Maximum total distance
     */
    long long maxDistance(vector<int>& arr, int K) {
        // Min-heap to store wind speeds of days we decided to sail.
        // The smallest element represents the "worst" sailing day so far.
        priority_queue<int, vector<int>, greater<int>> minHeap;
        
        long long totalDistance = 0;  // Total distance traveled
        int power = K;                // Current available power
        
        // Process days from left to right
        for (int wind : arr) {
            // CASE 1: We have power available -> we can sail
            if (power > 0) {
                // Sail today
                minHeap.push(wind);       // Remember this sailing day
                totalDistance += wind;    // Gain distance
                power--;                  // Spend 1 power
            }
            // CASE 2: No power available -> must decide carefully
            else {
                // Check if replacing a previous sailing day is beneficial
                if (!minHeap.empty() && minHeap.top() < wind) {
                    // Replace the worst previous sailing day with today
                    totalDistance -= minHeap.top(); // Remove old distance
                    minHeap.pop();                  // Remove worst day
                    
                    minHeap.push(wind);             // Add better day
                    totalDistance += wind;          // Add new distance
                    // Power remains 0 (we replaced, didn't gain power)
                }
                else {
                    // Not worth sailing today -> rest
                    power++;  // Resting regenerates power
                }
            }
        }
        
        return totalDistance;
    }
};
```

### Explanation
- **Line 18-19**: Initialize min-heap to track sailing days and variables:
  - **minHeap**: Stores wind speeds of days we sailed on (smallest = worst day)
  - **totalDistance**: Accumulated distance
  - **power**: Current available power
  
- **Line 21-42**: Process each day:
  - **Line 23-28**: **Case 1 - Have Power**:
    - Sail today (add wind to heap)
    - Add distance
    - Decrease power
    
  - **Line 29-42**: **Case 2 - No Power**:
    - **Line 31-37**: If today's wind > worst sailing day:
      - Remove worst day from heap (subtract its distance)
      - Add today (add its distance)
      - Replace worst with better
    - **Line 38-41**: Otherwise:
      - Rest and regain power
      
**Why Min-Heap?**
- Always gives us the **worst** sailing day in O(1) time
- Allows replacement in O(log n) time
- Maintains the K best days we've seen so far

**Example Walkthrough:**
```
arr = [5, 3, 7, 2], K = 2

Day 0 (wind=5):
  Power=2 > 0 → Sail
  Heap: [5], Distance=5, Power=1

Day 1 (wind=3):
  Power=1 > 0 → Sail
  Heap: [3, 5], Distance=8, Power=0

Day 2 (wind=7):
  Power=0, Heap.top()=3 < 7 → Replace
  Remove 3, add 7
  Heap: [5, 7], Distance=12, Power=0

Day 3 (wind=2):
  Power=0, Heap.top()=5 > 2 → Rest
  Heap: [5, 7], Distance=12, Power=1

Result: 12
```

## Complexity Analysis

### Time Complexity
- **O(n log K)**: 
  - Process n days
  - Each heap operation (push/pop) is O(log K)
  - In worst case, heap size is at most K
  - **Overall**: O(n log K)

### Space Complexity
- **O(K)**: 
  - Min-heap stores at most K elements
  - Other variables are O(1)
  - **Overall**: O(K)

## Key Insights
- **Greedy Strategy**: Always sail on the best K days seen so far
- **Min-Heap for Replacement**: Track worst sailing day to enable efficient replacement
- **Two Cases**: Have power (sail) vs. No power (replace or rest)
- **Replacement Logic**: Only replace if new day is better than worst current sailing day
- **No Upper Limit on Power**: Power can exceed K, but we only track K sailing days
- **Correctness**: Heap always contains the optimal set of sailing days at any point

## Alternative Approaches

### Dynamic Programming (Similar to Q1)

```cpp
long long maxDistanceDP(vector<int>& arr, int K) {
    int n = arr.size();
    // dp[day][power] = max distance from day to end with given power
    vector<vector<long long>> dp(n + 1, vector<long long>(K + n + 1, -1));
    
    function<long long(int, int)> solve = [&](int day, int power) -> long long {
        if (day == n) return 0;
        if (dp[day][power] != -1) return dp[day][power];
        
        long long rest = solve(day + 1, power + 1);
        long long sail = (power > 0) ? arr[day] + solve(day + 1, power - 1) : 0;
        
        return dp[day][power] = max(rest, sail);
    };
    
    return solve(0, K);
}
```

**Time Complexity:** O(n × (K + n))  
**Space Complexity:** O(n × (K + n))

**When to Use:**
- When power has an upper limit (capped at K)
- When we need exact optimal solution for all states
- Greedy approach is better when power can grow unbounded

### Comparison

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| **Greedy + Heap** | O(n log K) | O(K) | Power can grow, optimal for large n |
| **DP** | O(n × (K+n)) | O(n × (K+n)) | Power capped, need all states |

## Related Problems
- Maximum Distance with Energy Constraint (Q1) - DP approach
- Meeting Rooms II
- Task Scheduler
- Maximum Profit in Job Scheduling

