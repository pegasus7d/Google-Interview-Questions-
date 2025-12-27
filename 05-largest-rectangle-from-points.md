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

**Part 2 - Three Points Approach:**
- For a rotated rectangle, given 3 points, we can compute if they form a rectangle
- For each pair of points (potential diagonal), find all other points that form rectangles
- **Key Insight:** Use vector mathematics to check if points form a rectangle

### Step 3: Algorithm Design

**Part 1:**
1. Store all points in a hash set for O(1) lookup
2. For each pair of points `(x1, y1)` and `(x2, y2)`:
   - Check if `x1 != x2` and `y1 != y2` (not on same horizontal/vertical line)
   - Check if points `(x1, y2)` and `(x2, y1)` exist
   - If yes, calculate area = `|x2 - x1| × |y2 - y1|`
3. Return maximum area

**Part 2:**
1. For each pair of points (potential diagonal endpoints)
2. For each third point:
   - Check if these 3 points can form a rectangle
   - Calculate the 4th point using vector mathematics
   - Check if 4th point exists
   - Calculate area using cross product or determinant

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

#### Why String Encoding?

**Question:** Why use string encoding instead of simpler methods?

**Answer:** We need to encode `(x, y)` pairs for hash set lookup. Options:

1. **String encoding** `"x,y"`: Simple, works for any integer range
2. **Long long encoding** `(x << 32) | y`: More efficient but limited:
   - Requires `x` and `y` to fit in 32 bits
   - Can't handle negative numbers directly without offset
   - More complex to implement correctly

**Better Alternative - Using pair in unordered_set:**

```cpp
// Define hash function for pair<int, int>
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 1);
    }
};

unordered_set<pair<int, int>, PairHash> pointSet;
for (auto& p : points) {
    pointSet.insert(p);
}

// Then check:
pair<int, int> p3 = {x1, y2};
pair<int, int> p4 = {x2, y1};
if (pointSet.count(p3) && pointSet.count(p4)) {
    // Calculate area
}
```

**Complete Code with Pair Hash:**

```cpp
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <functional>
using namespace std;

// Hash function for pair<int, int>
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 1);
    }
};

class Solution {
public:
    double largestRectangleAxisAligned(vector<pair<int, int>>& points) {
        int n = points.size();
        if (n < 4) return 0.0;
        
        // Store points in hash set
        unordered_set<pair<int, int>, PairHash> pointSet;
        for (auto& p : points) {
            pointSet.insert(p);
        }
        
        double maxArea = 0.0;
        
        // Try all pairs as diagonal endpoints
        for (int i = 0; i < n; i++) {
            int x1 = points[i].first;
            int y1 = points[i].second;
            
            for (int j = i + 1; j < n; j++) {
                int x2 = points[j].first;
                int y2 = points[j].second;
                
                // Skip if on same line
                if (x1 == x2 || y1 == y2) continue;
                
                // Other two vertices
                pair<int, int> p3 = {x1, y2};
                pair<int, int> p4 = {x2, y1};
                
                // Check if both exist
                if (pointSet.count(p3) && pointSet.count(p4)) {
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

### Part 2: Rotated Rectangle

#### Logic Explanation

**Step-by-Step:**

1. **For each pair of points (potential diagonal):**
   - Points `A(x1, y1)` and `B(x2, y2)` could be diagonal endpoints

2. **For each third point `C(x3, y3)`:**
   - Check if `A`, `B`, `C` can form a rectangle
   - In a rectangle, if `AB` is a diagonal, then `C` and the 4th point `D` should satisfy:
     - `AC` and `BD` are the other diagonal, OR
     - `AC` and `BC` are adjacent sides

3. **Calculate 4th point `D`:**
   - If `A`, `B`, `C` form a rectangle, then:
     - Vector `AC` and vector `BC` should be perpendicular
     - The 4th point `D = A + (C - A) + (B - A) = B + C - A`

4. **Verify rectangle:**
   - Check if `D` exists in point set
   - Verify that `AC` ⟂ `BC` (dot product = 0)
   - Calculate area using cross product or determinant

**Why 3 Points?**
- Given 3 points of a rectangle, the 4th point is uniquely determined
- We can verify if the 4 points form a rectangle by checking:
  - Opposite sides are parallel
  - Adjacent sides are perpendicular

#### Code Implementation

```cpp
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <cmath>
#include <functional>
using namespace std;

// Hash function for pair<int, int>
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 1);
    }
};

class Solution {
private:
    /**
     * Calculate area of rectangle using determinant formula
     * Given 3 points A(x1,y1), B(x2,y2), C(x3,y3) forming a rectangle
     * Area = |det(AB, AC)| where AB and AC are vectors
     */
    double rectangleAreaUsingDeterminant(const pair<int, int>& A,
                                         const pair<int, int>& B,
                                         const pair<int, int>& C) {
        int x1 = A.first, y1 = A.second;
        int x2 = B.first, y2 = B.second;
        int x3 = C.first, y3 = C.second;
        
        // Vector AB: (x2-x1, y2-y1)
        // Vector AC: (x3-x1, y3-y1)
        // Area = |det(AB, AC)| = |(x2-x1)*(y3-y1) - (y2-y1)*(x3-x1)|
        
        double v1x = x2 - x1;
        double v1y = y2 - y1;
        double v2x = x3 - x1;
        double v2y = y3 - y1;
        
        // Determinant (cross product magnitude)
        double area = abs(v1x * v2y - v1y * v2x);
        return area;
    }
    
    /**
     * Check if two vectors are perpendicular (dot product = 0)
     */
    bool arePerpendicular(const pair<int, int>& A,
                         const pair<int, int>& B,
                         const pair<int, int>& C) {
        // Vector AB: (x2-x1, y2-y1)
        // Vector AC: (x3-x1, y3-y1)
        // Dot product = (x2-x1)*(x3-x1) + (y2-y1)*(y3-y1)
        
        int x1 = A.first, y1 = A.second;
        int x2 = B.first, y2 = B.second;
        int x3 = C.first, y3 = C.second;
        
        double dotProduct = (x2 - x1) * (x3 - x1) + (y2 - y1) * (y3 - y1);
        
        // Check if approximately zero (handling floating point)
        return abs(dotProduct) < 1e-9;
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
        
        // Store points in hash set for O(1) lookup
        unordered_set<pair<int, int>, PairHash> pointSet;
        for (auto& p : points) {
            pointSet.insert(p);
        }
        
        double maxArea = 0.0;
        
        // Try all pairs of points as potential diagonal endpoints
        for (int i = 0; i < n; i++) {
            const auto& A = points[i];  // First diagonal endpoint
            int x1 = A.first, y1 = A.second;
            
            for (int j = i + 1; j < n; j++) {
                const auto& B = points[j];  // Second diagonal endpoint
                int x2 = B.first, y2 = B.second;
                
                // For each third point
                for (int k = 0; k < n; k++) {
                    if (k == i || k == j) continue;
                    
                    const auto& C = points[k];  // Third point
                    int x3 = C.first, y3 = C.second;
                    
                    // Check if AB and AC are perpendicular (rectangle property)
                    // In a rectangle, if A, B, C are three vertices, then:
                    // Either AB ⟂ AC (A is a corner) or we need different check
                    
                    // Actually, for rectangle with vertices A, B, C, D:
                    // If A and B are diagonal endpoints, then C and D are the other two
                    // We need to check: AC ⟂ BC (C is a corner)
                    
                    // Vector AC: (x3-x1, y3-y1)
                    // Vector BC: (x3-x2, y3-y2)
                    double acx = x3 - x1;
                    double acy = y3 - y1;
                    double bcx = x3 - x2;
                    double bcy = y3 - y2;
                    
                    // Check if AC ⟂ BC (dot product = 0)
                    double dotProduct = acx * bcx + acy * bcy;
                    if (abs(dotProduct) > 1e-9) continue;  // Not perpendicular
                    
                    // Calculate 4th point D
                    // In rectangle, if A, B are diagonal and C is a vertex,
                    // then D = A + B - C (vector addition)
                    int x4 = x1 + x2 - x3;
                    int y4 = y1 + y2 - y3;
                    pair<int, int> D = {x4, y4};
                    
                    // Check if D exists and is different from A, B, C
                    if (pointSet.count(D) && D != A && D != B && D != C) {
                        // Calculate area using determinant
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
#include <functional>
using namespace std;

// Hash function for pair<int, int>
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 1);
    }
};

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
        
        unordered_set<pair<int, int>, PairHash> pointSet;
        for (auto& p : points) {
            pointSet.insert(p);
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
                pair<int, int> p3 = {x1, y2};
                pair<int, int> p4 = {x2, y1};
                
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
        
        unordered_set<pair<int, int>, PairHash> pointSet;
        for (auto& p : points) {
            pointSet.insert(p);
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
                    pair<int, int> D = {x4, y4};
                    
                    // Check if D exists and is distinct
                    if (pointSet.count(D) && D != A && D != B && D != C) {
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
| Part 2 | Three points | O(n³) | O(n) |

**Note:** Part 2 can be optimized further, but O(n³) is reasonable for n ≤ 1000.

## Key Insights
- **Diagonal Approach (Part 1)**: Instead of trying all 4 points O(n⁴), use 2 points as diagonal and check if other 2 exist O(n²)
- **Three Points Approach (Part 2)**: Given 3 points, the 4th point is uniquely determined if they form a rectangle
- **Perpendicular Check**: Use dot product = 0 to verify adjacent sides are perpendicular
- **Area Calculation**: Determinant/cross product works for rectangles at any angle
- **Hash Set**: Essential for O(1) point existence checking

## Related Problems
- Max Points on a Line
- Rectangle Overlap
- Largest Rectangle in Histogram
- Number of Rectangles That Can Form the Largest Square

