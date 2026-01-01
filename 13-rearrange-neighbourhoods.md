# Rearrange Neighbourhoods

## Problem Statement

You are given an array of arrays where each inner array represents a **neighbourhood** with house numbers. You need to rearrange houses such that:

1. **Houses in each neighbourhood are sorted** (in ascending order)
2. **Each neighbourhood size is maintained** (same number of houses as original)
3. **No neighbourhood has duplicate house numbers** (all houses in a neighbourhood are unique)
4. **Same house numbers can exist in multiple neighbourhoods** (houses can be shared across neighbourhoods)

**Input Format:**
- `neighbourhoods`: Vector of vectors, where each inner vector represents a neighbourhood with house numbers

**Output:** Return `true` if rearrangement is possible, `false` otherwise. The function should modify `neighbourhoods` in-place.

## Examples

### Example 1
```
Input:
neighbourhoods = [
    [3, 1, 2],
    [2, 4, 1],
    [1, 3]
]

After rearrangement:
neighbourhoods = [
    [1, 2, 3],  // sorted, size 3, no duplicates
    [1, 2, 4],  // sorted, size 3, no duplicates
    [1, 3]      // sorted, size 2, no duplicates
]

Output: true
```

### Example 2
```
Input:
neighbourhoods = [
    [1, 1, 2],
    [2, 2]
]

Total houses: 1, 1, 2, 2, 2 = 5 houses
Neighbourhood 1 needs 3 unique houses
Neighbourhood 2 needs 2 unique houses

We only have 2 distinct house numbers (1 and 2)
Cannot satisfy neighbourhood 1 which needs 3 unique houses

Output: false
```

### Example 3
```
Input:
neighbourhoods = [
    [5, 3, 1],
    [2, 4],
    [1, 2, 3, 4]
]

After rearrangement:
neighbourhoods = [
    [1, 3, 5],  // sorted, size 3
    [2, 4],     // sorted, size 2
    [1, 2, 3, 4] // sorted, size 4
]

Output: true
```

## Constraints
- `1 <= neighbourhoods.length <= 100`
- `1 <= neighbourhoods[i].length <= 100`
- `1 <= house numbers <= 10^4`
- Total number of houses across all neighbourhoods <= 10^4

## Approach

### Step 1: Understanding the Problem
We need to redistribute houses such that:
- Each neighbourhood has the same size as before
- Each neighbourhood contains only unique house numbers
- Each neighbourhood is sorted
- We can use the same house number in different neighbourhoods

**Key Insight:** We need to check if we have enough **distinct house numbers** to satisfy all neighbourhoods' requirements.

### Step 2: Identifying the Strategy
**Greedy Approach:**
1. Collect all house numbers in a multiset (sorted, allows duplicates)
2. For each neighbourhood (in order):
   - Clear the neighbourhood
   - Fill it with the smallest available house numbers that haven't been used in this neighbourhood
   - If we run out of houses, return false
3. Sort each neighbourhood after filling

**Why Multiset?**
- Keeps elements sorted automatically
- Allows duplicates (same house number can appear multiple times)
- Easy to remove smallest element

### Step 3: Algorithm Design
1. **Collect all houses**: Store all house numbers in a multiset
2. **Store original sizes**: Remember the size of each neighbourhood
3. **For each neighbourhood**:
   - Clear current neighbourhood
   - Create a set to track used house numbers in current neighbourhood
   - While neighbourhood size < original size:
     - Take smallest house from multiset
     - If not used in current neighbourhood, add it
     - If already used, skip and put it back (or track for later)
   - Sort the neighbourhood
4. **Return true** if all neighbourhoods filled successfully

### Step 4: Edge Cases
- Not enough distinct houses: If a neighbourhood needs more unique houses than available distinct values
- Not enough total houses: If total houses is less than sum of neighbourhood sizes
- Empty neighbourhoods: Handle gracefully
- All houses same: Need enough duplicates

## Solution

### Code Implementation

```cpp
#include <vector>
#include <set>
#include <unordered_set>
#include <algorithm>
using namespace std;

class Solution {
public:
    /**
     * Rearrange houses in neighbourhoods such that:
     * - Each neighbourhood is sorted
     * - Each neighbourhood size is maintained
     * - No neighbourhood has duplicate house numbers
     * 
     * @param neighbourhoods Vector of neighbourhoods (modified in-place)
     * @return true if rearrangement is possible, false otherwise
     */
    bool rearrangeNeighbourhoods(vector<vector<int>>& neighbourhoods) {
        int n = neighbourhoods.size();
        vector<int> sizes;
        multiset<int> st;  // Multiset to store all houses (sorted, allows duplicates)
        
        // Step 1: Collect all houses and store original sizes
        for (auto& neighbourhood : neighbourhoods) {
            sizes.push_back(neighbourhood.size());
            for (auto house : neighbourhood) {
                st.insert(house);
            }
        }
        
        // Step 2: Rearrange each neighbourhood
        for (int i = 0; i < n; i++) {
            neighbourhoods[i].clear();
            unordered_set<int> used;  // Track houses used in current neighbourhood
            vector<int> skipped;      // Houses skipped due to duplicates
            
            // Fill neighbourhood until it reaches original size
            while (neighbourhoods[i].size() < sizes[i]) {
                // If multiset is empty, we don't have enough houses
                if (st.empty()) {
                    return false;
                }
                
                // Take smallest house from multiset
                auto it = st.begin();
                int val = *it;
                st.erase(it);
                
                // If this house number is already used in current neighbourhood, skip it
                if (used.find(val) != used.end()) {
                    skipped.push_back(val);
                    continue;
                }
                
                // Add house to current neighbourhood
                neighbourhoods[i].push_back(val);
                used.insert(val);
            }
            
            // Put back skipped houses to multiset
            for (int x : skipped) {
                st.insert(x);
            }
            
            // Sort the neighbourhood
            sort(neighbourhoods[i].begin(), neighbourhoods[i].end());
        }
        
        return true;
    }
};
```

### Explanation
- **Line 20-27**: Collect all houses and store sizes:
  - **Line 22**: Store original size of each neighbourhood
  - **Line 23-25**: Insert all houses into multiset (automatically sorted)
  
- **Line 29-55**: Rearrange each neighbourhood:
  - **Line 30**: Clear current neighbourhood
  - **Line 31**: Create set to track used house numbers in current neighbourhood
  - **Line 32**: Create vector to store skipped houses
  - **Line 34-50**: Fill neighbourhood until it reaches original size:
    - **Line 36-39**: Check if we have enough houses
    - **Line 41-43**: Take smallest house from multiset
    - **Line 45-48**: If house number already used in current neighbourhood, skip it
    - **Line 50-52**: Otherwise, add to neighbourhood and mark as used
  - **Line 53-55**: Put back skipped houses to multiset for next neighbourhoods
  - **Line 57-58**: Sort the neighbourhood
  
**Why This Works:**
- **Multiset**: Keeps houses sorted, so we always take the smallest available
- **Used Set**: Ensures no duplicates within a neighbourhood
- **Skipped Houses**: Houses that can't be used in current neighbourhood are put back for other neighbourhoods
- **Sorting**: Ensures final neighbourhood is sorted (though multiset already gives sorted order)

## Complexity Analysis

### Time Complexity
- **O(N × M × log(M))**: 
  - `N` = number of neighbourhoods
  - `M` = average neighbourhood size
  - Collecting houses: O(total_houses × log(total_houses))
  - For each neighbourhood: O(size × log(total_houses)) for multiset operations
  - Sorting each neighbourhood: O(size × log(size))
  - **Overall**: O(N × M × log(M))

### Space Complexity
- **O(total_houses)**: 
  - Multiset: O(total_houses)
  - Used set per neighbourhood: O(M)
  - Skipped vector: O(M)
  - **Overall**: O(total_houses)

## Key Insights
- **Multiset for Sorting**: Using multiset automatically keeps houses sorted
- **Greedy Selection**: Always take smallest available house that hasn't been used in current neighbourhood
- **Duplicate Handling**: Skip duplicates within a neighbourhood, but allow same house in different neighbourhoods
- **Early Termination**: Return false if we run out of houses before filling a neighbourhood
- **Size Preservation**: Must maintain original size of each neighbourhood

## Alternative Approaches

### Using Priority Queue

```cpp
bool rearrangeNeighbourhoodsWithPQ(vector<vector<int>>& neighbourhoods) {
    int n = neighbourhoods.size();
    vector<int> sizes;
    priority_queue<int, vector<int>, greater<int>> pq;  // Min-heap
    
    // Collect all houses
    for (auto& neighbourhood : neighbourhoods) {
        sizes.push_back(neighbourhood.size());
        for (auto house : neighbourhood) {
            pq.push(house);
        }
    }
    
    for (int i = 0; i < n; i++) {
        neighbourhoods[i].clear();
        unordered_set<int> used;
        vector<int> skipped;
        
        while (neighbourhoods[i].size() < sizes[i]) {
            if (pq.empty()) return false;
            
            int val = pq.top();
            pq.pop();
            
            if (used.count(val)) {
                skipped.push_back(val);
                continue;
            }
            
            neighbourhoods[i].push_back(val);
            used.insert(val);
        }
        
        // Put back skipped houses
        for (int x : skipped) {
            pq.push(x);
        }
        
        sort(neighbourhoods[i].begin(), neighbourhoods[i].end());
    }
    
    return true;
}
```

**Time Complexity:** O(N × M × log(M))  
**Space Complexity:** O(total_houses)

### Pre-check Feasibility

```cpp
bool rearrangeNeighbourhoodsWithCheck(vector<vector<int>>& neighbourhoods) {
    int n = neighbourhoods.size();
    
    // Pre-check: Count distinct houses and total houses
    unordered_set<int> distinctHouses;
    int totalHouses = 0;
    vector<int> sizes;
    
    for (auto& neighbourhood : neighbourhoods) {
        sizes.push_back(neighbourhood.size());
        totalHouses += neighbourhood.size();
        for (auto house : neighbourhood) {
            distinctHouses.insert(house);
        }
    }
    
    // Check if any neighbourhood needs more unique houses than available
    for (int size : sizes) {
        if (size > distinctHouses.size()) {
            return false;  // Not enough distinct houses
        }
    }
    
    // Now proceed with rearrangement (same as before)
    multiset<int> st;
    for (auto& neighbourhood : neighbourhoods) {
        for (auto house : neighbourhood) {
            st.insert(house);
        }
    }
    
    // ... rest of the algorithm
    return true;
}
```

**Advantage:** Early detection of impossible cases

## Related Problems
- Meeting Rooms II
- Car Pooling
- Task Scheduler
- Reorganize String

