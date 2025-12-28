# Largest Square Submatrix of 1's

## Problem Statement

Given a binary matrix of size `M × N`, find the position (upper-left corner) of a square submatrix containing **all 1's** with size **greater than or equal to `k`**.

**Input Format:**
- `matrix`: Binary matrix (contains only 0's and 1's)
- `k`: Minimum required size of the square submatrix

**Output:** Return the position `{row, col}` of the upper-left corner of the square submatrix. If no such square exists, return `{-1, -1}`.

## Examples

### Example 1
```
Input:
matrix = [
    [1, 1, 1, 0],
    [1, 1, 1, 1],
    [1, 1, 1, 1],
    [0, 1, 1, 1]
]
k = 3

Largest square of 1's:
Starting at (1, 1):
[1, 1, 1]
[1, 1, 1]
[1, 1, 1]

Size = 3 >= k = 3
Output: {1, 1}
```

### Example 2
```
Input:
matrix = [
    [1, 1, 0],
    [1, 1, 1],
    [0, 1, 1]
]
k = 2

Largest square of 1's:
Starting at (0, 0):
[1, 1]
[1, 1]

Size = 2 >= k = 2
Output: {0, 0}
```

### Example 3
```
Input:
matrix = [
    [1, 0, 1],
    [0, 1, 0],
    [1, 0, 1]
]
k = 2

No square of size >= 2 exists
Output: {-1, -1}
```

## Constraints
- `1 <= M, N <= 1000`
- `matrix[i][j]` is either 0 or 1
- `1 <= k <= min(M, N)`

## Approach

### Step 1: Understanding the Problem
We need to find a square submatrix (contiguous) of all 1's with size at least `k`. The square must be:
- **Contiguous**: All cells are adjacent
- **Square**: Same number of rows and columns
- **All 1's**: Every cell in the square is 1
- **Size >= k**: The side length is at least `k`

### Step 2: Identifying the Strategy
This is a classic **Dynamic Programming** problem:
- For each position `(i, j)`, find the **largest square of 1's** starting at that position
- Use the recurrence: `dp[i][j] = 1 + min(dp[i][j+1], dp[i+1][j], dp[i+1][j+1])`
- If `dp[i][j] >= k`, return `{i, j}`

**Key Insight:** The largest square starting at `(i, j)` depends on the squares starting at its three neighbors (right, down, diagonal).

### Step 3: Algorithm Design
1. **DP State**: `dp[i][j]` = size of largest square of 1's starting at `(i, j)`
2. **Base Cases**:
   - If `matrix[i][j] == 0`: `dp[i][j] = 0`
   - If out of bounds: return 0
3. **Recurrence**:
   - `dp[i][j] = 1 + min(dp[i][j+1], dp[i+1][j], dp[i+1][j+1])`
   - Why min? Because the square can only extend as far as the smallest of the three neighbors
4. **Memoization**: Store results to avoid recomputation
5. **Search**: Check all positions, return first one with `dp[i][j] >= k`

### Step 4: Edge Cases
- No square of size >= k exists: Return `{-1, -1}`
- Matrix is all 0's: Return `{-1, -1}`
- Matrix is all 1's: Return `{0, 0}` if size >= k
- Multiple valid squares: Return the first one found (top-left to bottom-right)

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
private:
    int m, n;
    vector<vector<int>> memo;
    vector<vector<int>>* mat;
    
    /**
     * Find the size of largest square of 1's starting at position (i, j)
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
        
        // Base case: current cell is 0, cannot form square
        if ((*mat)[i][j] == 0) {
            return 0;
        }
        
        // If already computed, return memoized result
        if (memo[i][j] != -1) {
            return memo[i][j];
        }
        
        // Recursively compute squares starting at three neighbors
        int right = solve(i, j + 1);      // Square starting at (i, j+1)
        int down  = solve(i + 1, j);      // Square starting at (i+1, j)
        int diag  = solve(i + 1, j + 1);  // Square starting at (i+1, j+1)
        
        // The square at (i, j) can extend as far as the smallest neighbor
        // Add 1 for the current cell
        return memo[i][j] = 1 + min({right, down, diag});
    }
    
public:
    /**
     * Find position of square submatrix of all 1's with size >= k
     * 
     * @param matrix Binary matrix
     * @param k Minimum required size
     * @return {row, col} of upper-left corner, or {-1, -1} if not found
     */
    pair<int, int> findSquare(vector<vector<int>>& matrix, int k) {
        mat = &matrix;
        m = matrix.size();
        n = matrix[0].size();
        
        // Initialize memoization table
        memo.assign(m, vector<int>(n, -1));
        
        // Check each position as potential upper-left corner
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // Find largest square starting at (i, j)
                int squareSize = solve(i, j);
                
                // If square size is at least k, return position
                if (squareSize >= k) {
                    return {i, j};
                }
            }
        }
        
        // No square of size >= k found
        return {-1, -1};
    }
};
```

### Explanation
- **Line 18-20**: Store matrix dimensions and memoization table
- **Line 21**: Pointer to matrix (to avoid copying)
- **Line 28-50**: `solve` function (recursive DP with memoization):
  - **Line 31-33**: Base case: return 0 if out of bounds
  - **Line 35-38**: Base case: return 0 if current cell is 0
  - **Line 40-43**: Memoization check: return stored result if already computed
  - **Line 45-47**: Recursively compute squares starting at three neighbors:
    - `right`: Square starting at `(i, j+1)` (to the right)
    - `down`: Square starting at `(i+1, j)` (below)
    - `diag`: Square starting at `(i+1, j+1)` (diagonally down-right)
  - **Line 49-50**: Return `1 + min(right, down, diag)`:
    - Current cell contributes 1
    - Can only extend as far as the smallest neighbor square
    - Store in memo before returning
  
- **Line 55-75**: `findSquare` function:
  - **Line 57-59**: Initialize matrix pointer and dimensions
  - **Line 61-62**: Initialize memoization table with -1 (uncomputed)
  - **Line 64-72**: Check each position:
    - Compute largest square starting at `(i, j)`
    - If size >= k, return position immediately
  - **Line 74**: Return `{-1, -1}` if no valid square found

**Why `min(right, down, diag)`?**
- For a square to exist at `(i, j)`, all three neighbors must also be part of a square
- The square can only extend as far as the **smallest** of the three neighbor squares
- Example: If `right=2`, `down=3`, `diag=2`, then square at `(i, j)` can be at most `1 + min(2,3,2) = 3`

**Visual Example:**
```
If we have:
1 1 1
1 1 1
1 1 1

At position (0,0):
- right (0,1) = 2 (square of size 2x2)
- down (1,0) = 2
- diag (1,1) = 2
- Result: 1 + min(2,2,2) = 3 (square of size 3x3)
```

## Complexity Analysis

### Time Complexity
- **O(M × N)**: 
  - Each cell is computed exactly once due to memoization
  - Each computation takes O(1) time (just min of three values)
  - **Overall**: O(M × N)

### Space Complexity
- **O(M × N)**: 
  - Memoization table: O(M × N)
  - Recursion stack: O(M + N) in worst case (diagonal path)
  - **Overall**: O(M × N)

## Key Insights
- **DP Recurrence**: The size of square at `(i, j)` depends on three neighbors (right, down, diagonal)
- **Min Operation**: Use `min` because the square can only extend as far as the smallest neighbor
- **Memoization**: Essential to avoid exponential recomputation
- **Early Termination**: Return as soon as we find a square of size >= k
- **Bottom-Up Thinking**: We compute from bottom-right to top-left (though recursion goes top-left to bottom-right)

## Alternative Approaches

### Bottom-Up DP (Iterative)

```cpp
pair<int, int> findSquareIterative(vector<vector<int>>& matrix, int k) {
    int m = matrix.size();
    int n = matrix[0].size();
    
    // dp[i][j] = size of largest square starting at (i, j)
    vector<vector<int>> dp(m, vector<int>(n, 0));
    
    // Process from bottom-right to top-left
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            if (matrix[i][j] == 0) {
                dp[i][j] = 0;
            } else {
                // Get values from neighbors (with boundary checks)
                int right = (j + 1 < n) ? dp[i][j + 1] : 0;
                int down  = (i + 1 < m) ? dp[i + 1][j] : 0;
                int diag  = (i + 1 < m && j + 1 < n) ? dp[i + 1][j + 1] : 0;
                
                dp[i][j] = 1 + min({right, down, diag});
                
                // Check if this square meets requirement
                if (dp[i][j] >= k) {
                    return {i, j};
                }
            }
        }
    }
    
    return {-1, -1};
}
```

**Time Complexity:** O(M × N)  
**Space Complexity:** O(M × N)

**Advantages:**
- No recursion stack overhead
- More cache-friendly (sequential access)
- Easier to optimize space

### Space-Optimized DP (O(N) space)

```cpp
pair<int, int> findSquareSpaceOptimized(vector<vector<int>>& matrix, int k) {
    int m = matrix.size();
    int n = matrix[0].size();
    
    // Only store current and next row
    vector<int> prev(n, 0);
    vector<int> curr(n, 0);
    
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            if (matrix[i][j] == 0) {
                curr[j] = 0;
            } else {
                int right = (j + 1 < n) ? curr[j + 1] : 0;
                int down  = (i + 1 < m) ? prev[j] : 0;
                int diag  = (i + 1 < m && j + 1 < n) ? prev[j + 1] : 0;
                
                curr[j] = 1 + min({right, down, diag});
                
                if (curr[j] >= k) {
                    return {i, j};
                }
            }
        }
        swap(prev, curr);
    }
    
    return {-1, -1};
}
```

**Time Complexity:** O(M × N)  
**Space Complexity:** O(N) - only two rows

## Related Problems
- Maximal Square
- Largest Rectangle in Histogram
- Count Square Submatrices with All Ones
- Maximum Size Subarray Sum Equals k


