# Square of 1's with Minimum Side Length Constraint

## Problem Statement

Given a binary matrix of size `m × n` (not necessarily square) containing only 0's and 1's, find a **square submatrix of all 1's**.

**Constraints:**
- The square's side length must be **at least `⌈√n⌉`** (ceiling of square root of n)
- Return the **top-left corner** (row, col) and **side length** of the square
- If no such square exists, return `{0, 0, 0}`

**Input Format:**
- `matrix`: Binary matrix (contains only 0's and 1's)

**Output:** Vector `{row, col, side_length}` representing the top-left corner and side length of the square, or `{0, 0, 0}` if not found.

## Examples

### Example 1
```
Input:
matrix = [
    [0, 1, 1, 1],
    [1, 1, 1, 1],
    [1, 1, 1, 1],
    [0, 1, 1, 1]
]

n = 4, so minSide = ⌈√4⌉ = 2

Largest square of 1's:
Starting at (1, 1):
[1, 1]
[1, 1]

Size = 2 >= minSide = 2 ✓

Output: {1, 1, 2}
```

### Example 2
```
Input:
matrix = [
    [1, 1, 1, 1, 1],
    [1, 1, 1, 1, 1],
    [1, 1, 1, 1, 1]
]

n = 5, so minSide = ⌈√5⌉ = 3

Largest square of 1's:
Starting at (0, 0):
[1, 1, 1]
[1, 1, 1]
[1, 1, 1]

Size = 3 >= minSide = 3 ✓

Output: {0, 0, 3}
```

### Example 3
```
Input:
matrix = [
    [1, 0, 1],
    [0, 1, 0],
    [1, 0, 1]
]

n = 3, so minSide = ⌈√3⌉ = 2

No square of size >= 2 exists

Output: {0, 0, 0}
```

## Constraints
- `1 <= m, n <= 1000`
- `matrix[i][j]` is either 0 or 1
- Matrix is not necessarily square (m may not equal n)
- Minimum side length = `⌈√n⌉`

## Approach

### Step 1: Understanding the Problem
We need to find a square submatrix of all 1's with side length at least `⌈√n⌉`. This is similar to the "Maximal Square" problem but with a minimum size constraint.

**Key Insight:** Use dynamic programming to find the largest square starting at each position, then check if it meets the minimum size requirement.

### Step 2: Identifying the Strategy
**Dynamic Programming Approach:**
- For each position `(i, j)`, find the largest square of 1's starting at that position
- Use recurrence: `dp[i][j] = 1 + min(dp[i+1][j], dp[i][j+1], dp[i+1][j+1])`
- Check if `dp[i][j] >= minSide` and return the first valid square

**Why DP?**
- Overlapping subproblems: Same positions are checked multiple times
- Optimal substructure: Largest square at `(i, j)` depends on neighbors

### Step 3: Algorithm Design
1. **Calculate minimum side**: `minSide = ⌈√n⌉`
2. **Initialize DP table**: Memoization table for storing results
3. **For each position** `(i, j)`:
   - Compute largest square starting at `(i, j)` using recursion + memoization
   - If size >= minSide, return immediately
4. **Return result**: `{row, col, size}` or `{0, 0, 0}`

### Step 4: Edge Cases
- No square of size >= minSide exists: Return `{0, 0, 0}`
- Matrix is all 0's: Return `{0, 0, 0}`
- Matrix is all 1's: Return first position with size = min(m, n, minSide)
- Matrix dimensions smaller than minSide: May not be possible

## Solution

### Code Implementation

```cpp
#include <vector>
#include <cmath>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
private:
    // Global variables for recursion
    vector<vector<int>> mat;
    vector<vector<int>> memo;
    int m, n;
    
    /**
     * Find the largest square of 1's starting at position (i, j)
     * 
     * @param i Row index
     * @param j Column index
     * @return Size of largest square starting at (i, j)
     */
    int solve(int i, int j) {
        // Base case: out of bounds
        if (i >= m || j >= n) {
            return 0;
        }
        
        // If already computed, return memoized result
        if (memo[i][j] != -1) {
            return memo[i][j];
        }
        
        // Only if current cell is 1, try to grow a square
        if (mat[i][j] == 1) {
            // Recursively compute squares starting at three neighbors
            int down  = solve(i + 1, j);      // Square starting below
            int right = solve(i, j + 1);      // Square starting to the right
            int diag  = solve(i + 1, j + 1);  // Square starting diagonally
            
            // Current square can extend as far as the smallest neighbor
            return memo[i][j] = 1 + min({down, right, diag});
        }
        
        // If cell is 0, no square starts here
        return memo[i][j] = 0;
    }
    
public:
    /**
     * Find square of 1's with side length >= ⌈√n⌉
     * 
     * @param matrix Binary matrix
     * @return {row, col, side_length} or {0, 0, 0} if not found
     */
    vector<int> findSquareRecursive(vector<vector<int>>& matrix) {
        mat = matrix;
        m = mat.size();
        
        // Edge case: empty matrix
        if (m == 0) {
            return {0, 0, 0};
        }
        
        n = mat[0].size();
        
        // Calculate minimum side length: ⌈√n⌉
        int minSide = (int)ceil(sqrt(n));
        
        // Initialize memoization table
        memo.assign(m, vector<int>(n, -1));
        
        // Try all positions as potential top-left corner
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // Find largest square starting at (i, j)
                int size = solve(i, j);
                
                // If square meets minimum size requirement, return it
                if (size >= minSide) {
                    return {i, j, size};
                }
            }
        }
        
        // No valid square found
        return {0, 0, 0};
    }
};
```

### Explanation
- **Line 12-14**: Global variables to store matrix, memoization table, and dimensions
- **Line 20-42**: `solve` function (recursive DP with memoization):
  - **Line 23-25**: Base case: return 0 if out of bounds
  - **Line 27-30**: Memoization check: return stored result if already computed
  - **Line 32-39**: If current cell is 1:
    - **Line 34-36**: Recursively compute squares starting at three neighbors
    - **Line 38**: Return `1 + min(down, right, diag)` - current cell plus smallest neighbor
  - **Line 41-42**: If current cell is 0, return 0 (no square starts here)
  
- **Line 47-75**: `findSquareRecursive` function:
  - **Line 49-53**: Store matrix and get dimensions
  - **Line 55-57**: Handle empty matrix edge case
  - **Line 59**: Get number of columns
  - **Line 61-62**: Calculate minimum side length: `⌈√n⌉`
  - **Line 64-65**: Initialize memoization table with -1 (uncomputed)
  - **Line 67-76**: Try all positions:
    - **Line 70**: Compute largest square starting at `(i, j)`
    - **Line 72-75**: If size >= minSide, return immediately
  - **Line 78**: Return `{0, 0, 0}` if no valid square found

**Why `min(down, right, diag)`?**
- For a square to exist at `(i, j)`, all three neighbors must also be part of squares
- The square can only extend as far as the **smallest** of the three neighbor squares
- This ensures we form a valid square (all cells are 1's)

**Example:**
```
If at position (0,0):
- down = 2  (square of size 2x2 below)
- right = 3 (square of size 3x3 to right)
- diag = 2  (square of size 2x2 diagonally)

Then square at (0,0) = 1 + min(2,3,2) = 3
```

## Complexity Analysis

### Time Complexity
- **O(m × n)**: 
  - Each cell is computed exactly once due to memoization
  - Each computation takes O(1) time (min of three values)
  - **Overall**: O(m × n)

### Space Complexity
- **O(m × n)**: 
  - Memoization table: O(m × n)
  - Recursion stack: O(m + n) in worst case (diagonal path)
  - **Overall**: O(m × n)

## Key Insights
- **DP Recurrence**: Size of square at `(i, j)` = `1 + min(neighbors)`
- **Min Operation**: Square can only extend as far as smallest neighbor square
- **Memoization**: Essential to avoid exponential recomputation
- **Early Termination**: Return as soon as we find a square meeting the constraint
- **Minimum Side Constraint**: Only check squares with size >= `⌈√n⌉`
- **Global Variables**: Used to avoid passing matrix through recursion (cleaner code)

## Alternative Approaches

### Bottom-Up DP (Iterative)

```cpp
vector<int> findSquareIterative(vector<vector<int>>& matrix) {
    int m = matrix.size();
    if (m == 0) return {0, 0, 0};
    
    int n = matrix[0].size();
    int minSide = (int)ceil(sqrt(n));
    
    // dp[i][j] = size of largest square starting at (i, j)
    vector<vector<int>> dp(m, vector<int>(n, 0));
    
    // Process from bottom-right to top-left
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            if (matrix[i][j] == 0) {
                dp[i][j] = 0;
            } else {
                int down = (i + 1 < m) ? dp[i + 1][j] : 0;
                int right = (j + 1 < n) ? dp[i][j + 1] : 0;
                int diag = (i + 1 < m && j + 1 < n) ? dp[i + 1][j + 1] : 0;
                
                dp[i][j] = 1 + min({down, right, diag});
                
                // Check if meets minimum size requirement
                if (dp[i][j] >= minSide) {
                    return {i, j, dp[i][j]};
                }
            }
        }
    }
    
    return {0, 0, 0};
}
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(m × n)

**Advantages:**
- No recursion stack overhead
- More cache-friendly
- Can be space-optimized to O(n)

### Space-Optimized DP

```cpp
vector<int> findSquareSpaceOptimized(vector<vector<int>>& matrix) {
    int m = matrix.size();
    if (m == 0) return {0, 0, 0};
    
    int n = matrix[0].size();
    int minSide = (int)ceil(sqrt(n));
    
    // Only store current and next row
    vector<int> prev(n, 0);
    vector<int> curr(n, 0);
    
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            if (matrix[i][j] == 0) {
                curr[j] = 0;
            } else {
                int down = (i + 1 < m) ? prev[j] : 0;
                int right = (j + 1 < n) ? curr[j + 1] : 0;
                int diag = (i + 1 < m && j + 1 < n) ? prev[j + 1] : 0;
                
                curr[j] = 1 + min({down, right, diag});
                
                if (curr[j] >= minSide) {
                    return {i, j, curr[j]};
                }
            }
        }
        swap(prev, curr);
    }
    
    return {0, 0, 0};
}
```

**Time Complexity:** O(m × n)  
**Space Complexity:** O(n) - only two rows

## Related Problems
- Maximal Square
- Largest Square Submatrix
- Count Square Submatrices with All Ones
- Maximum Size Subarray Sum Equals k

