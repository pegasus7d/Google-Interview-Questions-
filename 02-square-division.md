# Horizontal Line Dividing Squares in Half

## Problem Statement

Given `n` squares in a 2D Cartesian plane where:
- Each square has its **top-left corner** at coordinates `(x, y)`
- Each square has **edge length** `a` (sides parallel to x and y axes)
- Squares can overlap

Find a **horizontal line** (parallel to x-axis) that divides the **total area** of all squares into two equal halves.

**Input Format:**
- `squares = [[x1, y1, a1], [x2, y2, a2], ...]` - Each element is `[x, y, edge_length]`

**Output:** The y-coordinate of the horizontal dividing line.

## Examples

### Example 1
```
Input: 
squares = [[1, 2, 5], [2, 4, 10]]

Square 1: top-left (1, 2), edge = 5
- Bottom edge at y = 2 - 5 = -3
- Top edge at y = 2
- Area = 25

Square 2: top-left (2, 4), edge = 10
- Bottom edge at y = 4 - 10 = -6
- Top edge at y = 4
- Area = 100

Total area = 125
Half area = 62.5

Output: The y-coordinate where area below = 62.5
```

### Example 2
```
Input:
squares = [[0, 0, 1], [2, 2, 1]]

Square 1: (0, 0), edge = 1
- Bottom: y = -1, Top: y = 0
- Area = 1

Square 2: (2, 2), edge = 1
- Bottom: y = 1, Top: y = 2
- Area = 1

Total area = 2
Half area = 1

Output: 1.0 (line at y=1 divides area equally)
```

### Example 3
```
Input:
squares = [[0, 5, 3], [0, 0, 2]]

Square 1: (0, 5), edge = 3
- Bottom: y = 2, Top: y = 5
- Area = 9

Square 2: (0, 0), edge = 2
- Bottom: y = -2, Top: y = 0
- Area = 4

Total area = 13
Half area = 6.5

Output: y-coordinate where area below = 6.5
```

## Constraints
- `1 <= n <= 10^4`
- `-10^9 <= x, y <= 10^9`
- `1 <= edge_length <= 10^9`
- Squares can overlap
- The dividing line always exists (total area > 0)
- Answer should be a floating point number

## Approach

### Step 1: Understanding the Problem
We need to find a horizontal line (y = constant) such that:
- Area of all squares **below** the line = Total area / 2
- Area of all squares **above** the line = Total area / 2

**Key Insight:** As we move a horizontal line upward, the area below it increases. We need to find the exact y-coordinate where this area equals half the total.

### Step 2: Identifying the Strategy
Instead of checking every possible y-coordinate (which would be inefficient), we use a **sweep line algorithm**:
- We only need to check **critical points** where squares start or stop contributing area (their bottom and top edges)
- Between these critical points, area increases **linearly** with height
- This reduces the problem from O(n log y) to O(n log n)

### Step 3: Algorithm Design
1. **Calculate Total Area**: Sum of all square areas (`edge²` for each square)
2. **Create Events**: For each square, create two events:
   - **Start Event**: (bottom_y, type=1, edge_length) - Square starts contributing area
   - **End Event**: (top_y, type=0, edge_length) - Square stops contributing area
3. **Sort Events**: Sort by y-coordinate to process from bottom to top
4. **Sweep Line**: Process events while tracking:
   - `combinedWidth`: Sum of edge lengths of squares currently intersected by the line
   - `currArea`: Accumulated area below current position
   - Between events, area increases at constant rate = `combinedWidth`
5. **Find Dividing Line**: When `currArea + areaDiff >= totalArea/2`, calculate exact position using linear interpolation:
   - `optimalHeightDiff = (totalArea/2 - currArea) / combinedWidth`
   - `answer = prevHeight + optimalHeightDiff`

### Step 4: Edge Cases
- Single square: Line divides that square in half
- Non-overlapping squares: Line might be at a square boundary
- Overlapping squares: Combined width changes at event points
- Squares at same y-coordinates: Events are processed in order
- When area exactly equals half at an event: Return that event's y-coordinate

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
#include <cmath>
using namespace std;

class Solution {
public:
    /**
     * Find horizontal line that divides total square area in half
     * 
     * @param squares Vector of [x, y, edge_length] for each square
     * @return y-coordinate of dividing line
     */
    double separateSquares(vector<vector<int>>& squares) {
        // Calculate total area
        long long totalArea = 0;
        for (auto& sq : squares) {
            long long edge = sq[2];
            totalArea += edge * edge;
        }
        
        if (totalArea == 0) return 0.0;
        
        // Create events: (y-coordinate, {type, edge_length})
        // type: 1 = square starts (bottom edge), 0 = square ends (top edge)
        vector<pair<double, pair<int, int>>> events;
        
        for (auto& sq : squares) {
            int y = sq[1], edge = sq[2];
            double bottomY = y - edge;  // Bottom edge of square
            double topY = y;            // Top edge of square
            
            // Event when square starts contributing (bottom edge)
            events.push_back({bottomY, {1, edge}});
            // Event when square stops contributing (top edge)
            events.push_back({topY, {0, edge}});
        }
        
        // Sort events by y-coordinate
        sort(events.begin(), events.end());
        
        // Initialize sweep line variables
        int combinedWidth = 0;      // Sum of edge lengths of active squares
        double currArea = 0.0;      // Area accumulated below current position
        double prevHeight = events[0].first;  // Previous event's y-coordinate
        
        // Process events from bottom to top, starting from index 1
        for (int i = 1; i < events.size(); i++) {
            // Update combined width based on previous event (events[i-1])
            int prevType = events[i-1].second.first;
            int prevEdge = events[i-1].second.second;
            if (prevType == 1) {
                combinedWidth += prevEdge;  // Previous square started contributing
            } else {
                combinedWidth -= prevEdge;  // Previous square stopped contributing
            }
            
            double currHeight = events[i].first;
            int type = events[i].second.first;   // 1 = start, 0 = end
            int edge = events[i].second.second;
            
            // Calculate height difference since last event
            double heightDiff = currHeight - prevHeight;
            
            // Area added between prevHeight and currHeight
            // Area = width × height (since width is constant in this interval)
            double areaDiff = combinedWidth * heightDiff;
            
            // Check if adding this area would exceed half total area
            if (currArea + areaDiff >= totalArea / 2.0) {
                // If exactly equal, return current height
                if (abs(currArea + areaDiff - totalArea / 2.0) < 1e-9) {
                    return currHeight;
                }
                // Otherwise, interpolate to find exact position
                if (combinedWidth == 0) {
                    return currHeight;  // Avoid division by zero
                }
                // Solve: currArea + (combinedWidth × optimalHeightDiff) = totalArea / 2
                double optimalHeightDiff = (totalArea / 2.0 - currArea) / combinedWidth;
                return prevHeight + optimalHeightDiff;
            }
            
            // Update accumulated area and previous height
            currArea += areaDiff;
            prevHeight = currHeight;
        }
        
        // Should never reach here, but return last height if needed
        return events.back().first;
    }
};
```

### Explanation
- **Line 15-20**: Calculate total area by summing `edge²` for each square
- **Line 22**: Handle edge case where total area is 0
- **Line 24-35**: Create events for each square:
  - Bottom edge event (type=1): Square starts contributing area
  - Top edge event (type=0): Square stops contributing area
- **Line 38**: Sort events by y-coordinate to process from bottom to top
- **Line 40-43**: Initialize sweep line variables:
  - `combinedWidth`: Tracks total width of squares currently intersected
  - `currArea`: Accumulated area below current position
  - `prevHeight`: Y-coordinate of previous event (first event's y-coordinate)
- **Line 45-75**: Process events starting from index 1:
  - **Line 47-52**: Update `combinedWidth` based on previous event (events[i-1]) before calculating area
  - **Line 54-56**: Get current event details
  - **Line 59**: Calculate height difference since last event
  - **Line 62**: Calculate area added in this interval = `width × height`
  - **Line 65-76**: If adding this area exceeds half total, find exact position:
    - If exactly equal, return current height
    - Otherwise, use linear interpolation to find exact position
  - **Line 78-79**: Update accumulated area and previous height

**Key Formula:**
```
optimalHeightDiff = (totalArea/2 - currArea) / combinedWidth
answer = prevHeight + optimalHeightDiff
```

## Complexity Analysis

### Time Complexity
- **O(n log n)**: Sorting `2n` events takes O(n log n) time
- **O(n)**: Single pass through events to find dividing line
- **Overall**: O(n log n)

### Space Complexity
- **O(n)**: Storing `2n` events (each square creates 2 events)
- **Overall**: O(n)

## Key Insights
- **Sweep Line Efficiency**: Instead of checking every y-coordinate, we only check critical points (square boundaries), reducing complexity from O(n log y) to O(n log n)
- **Linear Area Growth**: Between events, area increases linearly with height, allowing exact calculation via interpolation
- **Event-Based Processing**: Start/end events track when squares become active/inactive, maintaining `combinedWidth` efficiently
- **Combined Width**: Tracks total width of active squares, which determines the rate at which area increases
- **Why Not Binary Search**: Binary search would be O(n log y) where y can be very large (up to 10^9), making log y much larger than log n

## Alternative Approaches

### Binary Search Approach (O(n log y))
```cpp
double separateSquaresBinarySearch(vector<vector<int>>& squares) {
    // Calculate total area
    long long totalArea = 0;
    for (auto& sq : squares) {
        totalArea += (long long)sq[2] * sq[2];
    }
    
    // Find min and max y coordinates
    double minY = 1e18, maxY = -1e18;
    for (auto& sq : squares) {
        int y = sq[1], edge = sq[2];
        minY = min(minY, (double)(y - edge));
        maxY = max(maxY, (double)y);
    }
    
    // Binary search for dividing line
    double left = minY, right = maxY;
    for (int iter = 0; iter < 100; iter++) {  // 100 iterations for precision
        double mid = (left + right) / 2.0;
        long long areaBelow = calculateAreaBelow(squares, mid);
        
        if (areaBelow * 2 < totalArea) {
            left = mid;
        } else {
            right = mid;
        }
    }
    
    return (left + right) / 2.0;
}

long long calculateAreaBelow(vector<vector<int>>& squares, double y) {
    long long area = 0;
    for (auto& sq : squares) {
        int sqY = sq[1], edge = sq[2];
        double bottom = sqY - edge;
        double top = sqY;
        
        if (y >= top) {
            area += edge * edge;  // Entire square below
        } else if (y > bottom) {
            double height = y - bottom;
            area += edge * height;  // Partial square below
        }
    }
    return area;
}
```

**Time Complexity:** O(n log y) where y is the range of y-coordinates  
**Space Complexity:** O(1)

**Note:** This approach is simpler but slower when y-range is large, as log y can be much larger than log n.

## Related Problems
- Skyline Problem
- Rectangle Area II (Union of Rectangles)
- Meeting Rooms II
- Car Pooling
- The Skyline Problem
