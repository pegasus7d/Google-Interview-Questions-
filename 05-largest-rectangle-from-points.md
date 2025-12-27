# Largest Rectangle from Points

## Problem Statement

Given a set of points in a 2D plane, find **4 points** that form a rectangle with the **largest area**.

**Part 1:** Rectangle axes are **parallel to x and y axes** (axis-aligned rectangle)

**Part 2:** Rectangle axes are **not parallel to x and y axes** (rotated rectangle)

**Input Format:**
- `points`: Vector of `{x, y}` coordinates

**Output:** Maximum area of any rectangle formed by 4 points from the given set.

## Examples

### Part 1: Axis-Aligned Rectangle

```
Input: points = [(0,0), (2,0), (0,2), (2,2), (1,1), (3,3)]

Axis-aligned rectangles:
- Rectangle 1: (0,0), (2,0), (0,2), (2,2) → Area = 2 × 2 = 4
- Rectangle 2: (0,0), (2,0), (0,2), (2,2) is the only valid rectangle

Output: 4
```

### Part 2: Rotated Rectangle

```
Input: points = [(0,0), (1,1), (2,0), (1,-1)]

Rotated rectangle:
- Points: (0,0), (1,1), (2,0), (1,-1) form a square rotated 45°
- Area = √2 × √2 = 2

Output: 2.0
```

## Constraints
- `4 <= points.size() <= 1000`
- `-10^4 <= x, y <= 10^4`
- All points are distinct
- For Part 1: Rectangle sides are parallel to axes
- For Part 2: Rectangle can be rotated at any angle

## Approach

### Step 1: Understanding the Problem

**Part 1 (Axis-Aligned):**
- Rectangle has sides parallel to x and y axes
- If we have points `(x1, y1)` and `(x2, y2)` as diagonal endpoints, the other two points must be `(x1, y2)` and `(x2, y1)`
- We need to check if these 4 points exist

**Part 2 (Rotated):**
- Rectangle can be at any angle
- For a rectangle, opposite sides are parallel and equal, and diagonals bisect each other
- Given 3 points, we can compute the 4th point if they form a rectangle

### Step 2: Identifying the Strategy

**Part 1 - Diagonal Approach:**
- For any rectangle with axes parallel to axes, if we take two points as diagonal endpoints `(x1, y1)` and `(x2, y2)`, the other two vertices must be `(x1, y2)` and `(x2, y1)`
- **Key Insight:** Instead of trying all 4 points (O(n⁴)), we can:
  1. Try all pairs of points as diagonal endpoints: O(n²)
  2. Check if the other two vertices exist: O(1) with hash set
  3. Calculate area: O(1)

**Part 2 - Diagonal Grouping Approach:**
- **Key Insight:** In a rectangle, the two diagonals have:
  - The same midpoint
  - The same length
- **Optimization:** Group all point pairs (potential diagonals) by their midpoint and squared length
- For each group with at least 2 diagonals, any pair of diagonals can form a rectangle
- This reduces complexity by only checking diagonals that can actually form rectangles

### Step 3: Algorithm Design

**Part 1:**
1. Store all points in a hash set for O(1) lookup
2. For each pair of points `(x1, y1)` and `(x2, y2)`:
   - Check if `x1 != x2` and `y1 != y2` (not on same horizontal/vertical line)
   - Check if points `(x1, y2)` and `(x2, y1)` exist
   - If yes, calculate area = `|x2 - x1| × |y2 - y1|`
3. Return maximum area

**Part 2:**
1. **Group diagonals**: For each pair of points, calculate:
   - Midpoint: `((x1+x2)/2, (y1+y2)/2)`
   - Squared length: `(x1-x2)² + (y1-y2)²`
   - Create key from midpoint and squared length
2. **Store in map**: Group all diagonals with same key together
3. **Process groups**: For each group with at least 2 diagonals:
   - Try all pairs of diagonals in the group
   - Calculate area using determinant formula
   - Update maximum area

### Step 4: Edge Cases
- Less than 4 points: No rectangle possible
- All points collinear: No rectangle possible
- Multiple rectangles with same area: Return maximum
- Points with same x or y coordinates in Part 1: Skip (can't form diagonal)

## Solution

### Part 1: Axis-Aligned Rectangle

#### Code Implementation

```cpp
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <cmath>
#include <string>
using namespace std;

class Solution {
public:
    /**
     * Find largest axis-aligned rectangle from given points
     * 
     * @param points Vector of {x, y} coordinates
     * @return Maximum area of rectangle, or 0 if no rectangle exists
     */
    double largestRectangleAxisAligned(vector<pair<int, int>>& points) {
        int n = points.size();
        if (n < 4) return 0.0;
        
        // Store points in hash set for O(1) lookup
        // Using string encoding: "x,y"
        unordered_set<string> pointSet;
        for (auto& p : points) {
            pointSet.insert(to_string(p.first) + "," + to_string(p.second));
        }
        
        double maxArea = 0.0;
        
        // Try all pairs of points as diagonal endpoints
        for (int i = 0; i < n; i++) {
            int x1 = points[i].first;
            int y1 = points[i].second;
            
            for (int j = i + 1; j < n; j++) {
                int x2 = points[j].first;
                int y2 = points[j].second;
                
                // Skip if points are on same horizontal or vertical line
                // (they can't be diagonal endpoints of axis-aligned rectangle)
                if (x1 == x2 || y1 == y2) continue;
                
                // For diagonal (x1,y1) and (x2,y2), the other two vertices are:
                // (x1, y2) and (x2, y1)
                string p3 = to_string(x1) + "," + to_string(y2);
                string p4 = to_string(x2) + "," + to_string(y1);
                
                // Check if both points exist
                if (pointSet.count(p3) && pointSet.count(p4)) {
                    // Calculate area: width × height
                    double width = abs(x2 - x1);
                    double height = abs(y2 - y1);
                    double area = width * height;
                    maxArea = max(maxArea, area);
                }
            }
        }
        
        return maxArea;
    }
};
```

**Why String Encoding?**
- Simple and straightforward
- Works for any integer range (including negatives)
- No need for custom hash functions
- Easy to implement and understand


### Part 2: Rotated Rectangle

#### Logic Explanation

**Key Insight - Diagonal Properties:**
In a rectangle, the two diagonals have:
- **Same midpoint**: The diagonals bisect each other
- **Same length**: Both diagonals are equal

**Step-by-Step:**

1. **Group diagonals by midpoint and length:**
   - For each pair of points `(i, j)`, calculate:
     - Midpoint: `((x_i + x_j)/2, (y_i + y_j)/2)`
     - Squared length: `(x_i - x_j)² + (y_i - y_j)²`
   - Create a unique key from midpoint and squared length
   - Group all diagonals with the same key together

2. **Why this works:**
   - If two diagonals share the same midpoint and length, they can form a rectangle
   - The four endpoints of these two diagonals are the four vertices of the rectangle

3. **Calculate area:**
   - For each pair of diagonals in the same group:
     - Take 3 points: one endpoint from first diagonal, both endpoints from second diagonal
     - Use determinant formula to calculate area: `|det(AB, AC)|`

**Why Diagonal Grouping?**
- **Efficiency**: Instead of checking all O(n³) combinations, we only check diagonals that can actually form rectangles
- **Correctness**: Two diagonals with same midpoint and length guarantee a rectangle
- **Reduction**: Groups typically have few diagonals, making the inner loop efficient

#### Code Implementation

```cpp
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <cmath>
#include <string>
using namespace std;

class Solution {
private:
    /**
     * Build a unique key using midpoint and squared diagonal length.
     * In a rectangle, diagonals have the same midpoint and same length.
     */
    string makeKey(double midX, double midY, double lenSq) {
        return to_string(midX) + "#" +
               to_string(midY) + "#" +
               to_string(lenSq);
    }
    
    /**
     * Compute maximum rectangle area from one diagonal group.
     * All diagonals in this group share the same midpoint and length,
     * so they can form rectangles with each other.
     */
    double computeMaxAreaFromDiagonalGroup(
        const vector<pair<int, int>>& diagonals,
        const vector<pair<int, int>>& points
    ) {
        double maxArea = 0.0;
        int m = diagonals.size();
        
        // Try all pairs of diagonals in this group
        for (int i = 0; i < m; i++) {
            for (int j = i + 1; j < m; j++) {
                int a = diagonals[i].first;   // First point of diagonal i
                int b = diagonals[i].second;   // Second point of diagonal i
                int c = diagonals[j].first;    // First point of diagonal j
                
                // Get coordinates
                double x1 = points[a].first;
                double y1 = points[a].second;
                
                double x2 = points[b].first;
                double y2 = points[b].second;
                
                double x3 = points[c].first;
                double y3 = points[c].second;
                
                // Determinant formula for area
                // Area = |det(AB, AC)| where A=(x1,y1), B=(x2,y2), C=(x3,y3)
                double area = abs(
                    (x2 - x1) * (y3 - y1) -
                    (y2 - y1) * (x3 - x1)
                );
                
                maxArea = max(maxArea, area);
            }
        }
        
        return maxArea;
    }
    
public:
    /**
     * Find largest rotated rectangle from given points
     * 
     * @param points Vector of {x, y} coordinates
     * @return Maximum area of rectangle, or 0 if no rectangle exists
     */
    double largestRectangleRotated(vector<pair<int, int>>& points) {
        int n = points.size();
        if (n < 4) return 0.0;
        
        double maxArea = 0.0;
        
        // Group diagonals by (midpoint, squared length)
        // Key insight: In a rectangle, diagonals have same midpoint and same length
        unordered_map<string, vector<pair<int, int>>> diagonalGroups;
        
        // Enumerate all pairs of points as potential diagonals
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                // Calculate midpoint
                double midX = (points[i].first + points[j].first) / 2.0;
                double midY = (points[i].second + points[j].second) / 2.0;
                
                // Calculate squared diagonal length
                double dx = points[i].first - points[j].first;
                double dy = points[i].second - points[j].second;
                double lengthSquared = dx * dx + dy * dy;
                
                // Create key and group diagonals
                string key = makeKey(midX, midY, lengthSquared);
                diagonalGroups[key].push_back({i, j});
            }
        }
        
        // Compute rectangle areas from each group
        for (auto& entry : diagonalGroups) {
            const auto& diagonals = entry.second;
            
            // Need at least 2 diagonals to form a rectangle
            if (diagonals.size() < 2) continue;
            
            // All diagonals in this group can form rectangles with each other
            maxArea = max(
                maxArea,
                computeMaxAreaFromDiagonalGroup(diagonals, points)
            );
        }
        
        return maxArea;
    }
};
```

#### Why Cross Product / Determinant?

**Question:** Why use cross product/determinant instead of simpler methods?

**Answer:** 
- **Cross product** gives the area of parallelogram formed by two vectors
- For a rectangle, area = magnitude of cross product of two adjacent sides
- **Determinant formula** is the same as cross product magnitude:
  - `Area = |det(AB, AC)| = |(x2-x1)*(y3-y1) - (y2-y1)*(x3-x1)|`
- This works for rectangles at any angle, not just axis-aligned

**Alternative (Simpler but Less General):**
- For axis-aligned: `Area = width × height`
- For rotated: Need to calculate side lengths using distance formula, then multiply
- But cross product is more elegant and works universally

#### Complete Optimized Code

```cpp
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <cmath>
#include <string>
using namespace std;

class Solution {
private:
    /**
     * Calculate area of rectangle using determinant formula
     * Given 3 points A, B, C forming a rectangle
     * Area = |det(AB, AC)| where AB and AC are vectors from A
     */
    double rectangleAreaUsingDeterminant(const pair<int, int>& A,
                                         const pair<int, int>& B,
                                         const pair<int, int>& C) {
        int x1 = A.first, y1 = A.second;
        int x2 = B.first, y2 = B.second;
        int x3 = C.first, y3 = C.second;
        
        // Vector AB: (x2-x1, y2-y1)
        // Vector AC: (x3-x1, y3-y1)
        // Determinant = (x2-x1)*(y3-y1) - (y2-y1)*(x3-x1)
        
        double v1x = x2 - x1;
        double v1y = y2 - y1;
        double v2x = x3 - x1;
        double v2y = y3 - y1;
        
        double area = abs(v1x * v2y - v1y * v2x);
        return area;
    }
    
public:
    /**
     * Part 1: Largest axis-aligned rectangle
     */
    double largestRectangleAxisAligned(vector<pair<int, int>>& points) {
        int n = points.size();
        if (n < 4) return 0.0;
        
        // Store points in hash set using string encoding
        unordered_set<string> pointSet;
        for (auto& p : points) {
            pointSet.insert(to_string(p.first) + "," + to_string(p.second));
        }
        
        double maxArea = 0.0;
        
        // Try all pairs as diagonal endpoints
        for (int i = 0; i < n; i++) {
            int x1 = points[i].first, y1 = points[i].second;
            
            for (int j = i + 1; j < n; j++) {
                int x2 = points[j].first, y2 = points[j].second;
                
                // Skip if on same line
                if (x1 == x2 || y1 == y2) continue;
                
                // Other two vertices
                string p3 = to_string(x1) + "," + to_string(y2);
                string p4 = to_string(x2) + "," + to_string(y1);
                
                if (pointSet.count(p3) && pointSet.count(p4)) {
                    double area = abs(x2 - x1) * abs(y2 - y1);
                    maxArea = max(maxArea, area);
                }
            }
        }
        
        return maxArea;
    }
    
    /**
     * Part 2: Largest rotated rectangle
     */
    double largestRectangleRotated(vector<pair<int, int>>& points) {
        int n = points.size();
        if (n < 4) return 0.0;
        
        // Store points in hash set using string encoding
        unordered_set<string> pointSet;
        for (auto& p : points) {
            pointSet.insert(to_string(p.first) + "," + to_string(p.second));
        }
        
        double maxArea = 0.0;
        
        // Try all pairs as potential diagonal endpoints
        for (int i = 0; i < n; i++) {
            const auto& A = points[i];
            int x1 = A.first, y1 = A.second;
            
            for (int j = i + 1; j < n; j++) {
                const auto& B = points[j];
                int x2 = B.first, y2 = B.second;
                
                // For each third point
                for (int k = 0; k < n; k++) {
                    if (k == i || k == j) continue;
                    
                    const auto& C = points[k];
                    int x3 = C.first, y3 = C.second;
                    
                    // Check if AC ⟂ BC (C is a corner of rectangle)
                    double acx = x3 - x1;
                    double acy = y3 - y1;
                    double bcx = x3 - x2;
                    double bcy = y3 - y2;
                    
                    double dotProduct = acx * bcx + acy * bcy;
                    if (abs(dotProduct) > 1e-9) continue;  // Not perpendicular
                    
                    // Calculate 4th point: D = A + B - C
                    int x4 = x1 + x2 - x3;
                    int y4 = y1 + y2 - y3;
                    string D = to_string(x4) + "," + to_string(y4);
                    
                    // Check if D exists and is distinct
                    string A_str = to_string(x1) + "," + to_string(y1);
                    string B_str = to_string(x2) + "," + to_string(y2);
                    string C_str = to_string(x3) + "," + to_string(y3);
                    
                    if (pointSet.count(D) && D != A_str && D != B_str && D != C_str) {
                        double area = rectangleAreaUsingDeterminant(A, B, C);
                        maxArea = max(maxArea, area);
                    }
                }
            }
        }
        
        return maxArea;
    }
};
```

### Explanation

**Part 1 - Axis-Aligned:**
- **Line 67-70**: Store all points in hash set for O(1) lookup
- **Line 72-88**: Try all pairs `(i, j)` as diagonal endpoints
- **Line 78**: Skip if points are on same horizontal/vertical line
- **Line 81-82**: Calculate other two vertices: `(x1, y2)` and `(x2, y1)`
- **Line 84-87**: If both vertices exist, calculate and update maximum area

**Part 2 - Rotated:**
- **Line 95-99**: Store all points in hash set
- **Line 101-135**: Try all combinations:
  - **Line 103-108**: Try all pairs `(i, j)` as potential diagonal endpoints
  - **Line 110-115**: For each third point `k`
  - **Line 117-123**: Check if vectors `AC` and `BC` are perpendicular (dot product = 0)
  - **Line 125-126**: Calculate 4th point: `D = A + B - C`
  - **Line 128-131**: If `D` exists and is distinct, calculate area using determinant

**Why This Works:**
- In a rectangle, if `A` and `B` are diagonal endpoints and `C` is a vertex, then:
  - `AC` ⟂ `BC` (adjacent sides are perpendicular)
  - The 4th point `D` is uniquely determined: `D = A + B - C`
- The determinant formula gives the area of the parallelogram, which equals the rectangle area

## Complexity Analysis

### Part 1: Axis-Aligned Rectangle

**Time Complexity:**
- **O(n²)**: Try all pairs of points as diagonal endpoints
- **O(1)**: Hash set lookup for checking if points exist
- **Overall**: O(n²)

**Space Complexity:**
- **O(n)**: Hash set to store all points
- **Overall**: O(n)

### Part 2: Rotated Rectangle

**Time Complexity:**
- **O(n³)**: 
  - O(n²) for all pairs of points (diagonal endpoints)
  - O(n) for each third point
  - O(1) for hash set lookup and area calculation
- **Overall**: O(n³)

**Space Complexity:**
- **O(n)**: Hash set to store all points
- **Overall**: O(n)

### Comparison

| Part | Approach | Time Complexity | Space Complexity |
|------|----------|----------------|------------------|
| Part 1 | Diagonal (2 points) | O(n²) | O(n) |
| Part 2 | Diagonal grouping | O(n² + k²) | O(n²) |

**Note:** Part 2 can be optimized further, but O(n³) is reasonable for n ≤ 1000.

## Key Insights
- **Diagonal Approach (Part 1)**: Instead of trying all 4 points O(n⁴), use 2 points as diagonal and check if other 2 exist O(n²)
- **Diagonal Grouping (Part 2)**: Group diagonals by midpoint and length - only diagonals in same group can form rectangles
- **Rectangle Property**: In a rectangle, diagonals have same midpoint and same length
- **Area Calculation**: Determinant formula `|det(AB, AC)|` works for rectangles at any angle
- **Optimization**: Diagonal grouping significantly reduces the number of combinations to check

## Related Problems
- Max Points on a Line
- Rectangle Overlap
- Largest Rectangle in Histogram
- Number of Rectangles That Can Form the Largest Square

