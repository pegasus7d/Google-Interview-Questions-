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
using namespace std;

class Solution {
public:
    /**
     * Find horizontal line that divides total square area in half
     * 
     * @param squares Vector of [x, y, edge_length] for each square
     *                 where (x, y) is the top-left corner
     * @return y-coordinate of dividing line
     */
    double separateSquares(vector<vector<int>>& squares) {
        vector<pair<double, double>> events;
        double totalArea = 0.0;

        // Build events
        // For a square with top-left at (x, y) and edge length a:
        // - Bottom edge is at y - a (square starts contributing)
        // - Top edge is at y (square stops contributing)
        for (auto &sq : squares) {
            double y = sq[1];  // Top-left y-coordinate
            double a = sq[2];  // Edge length

            totalArea += a * a;
            events.push_back({y - a, +a}); // Square starts (bottom edge)
            events.push_back({y,     -a}); // Square ends (top edge)
        }

        sort(events.begin(), events.end());

        double target = totalArea / 2.0;
        double currArea = 0.0;
        double combinedWidth = 0.0;

        double prevY = events[0].first;
        combinedWidth += events[0].second;  // Initialize with first event

        for (int i = 1; i < events.size(); i++) {
            double currY = events[i].first;
            double heightDiff = currY - prevY;
            double areaAdded = combinedWidth * heightDiff;

            if (currArea + areaAdded >= target) {
                double deltaY = (target - currArea) / combinedWidth;
                return prevY + deltaY;
            }

            currArea += areaAdded;
            combinedWidth += events[i].second;  // Update width for next interval
            prevY = currY;
        }

        return -1.0;  // Should never reach here
    }
};
```

### Explanation
- **Line 13-23**: Create events for each square:
  - For a square with top-left at `(x, y)` and edge length `a`:
    - Bottom edge is at `y - a` (square starts contributing) → event `{y - a, +a}`
    - Top edge is at `y` (square stops contributing) → event `{y, -a}`
  - Calculate total area by summing `a²` for each square
- **Line 25**: Sort events by y-coordinate to process from bottom to top
- **Line 27-30**: Initialize sweep line variables:
  - `target`: Half of total area
  - `currArea`: Accumulated area below current position
  - `combinedWidth`: Sum of edge lengths of active squares
  - `prevY`: Y-coordinate of previous event
- **Line 31-32**: Process first event to initialize `combinedWidth`
- **Line 34-47**: Process remaining events starting from index 1:
  - **Line 36**: Calculate height difference since last event
  - **Line 37**: Calculate area added in this interval = `combinedWidth × heightDiff`
  - **Line 39-42**: If adding this area exceeds target, find exact position using linear interpolation:
    - `deltaY = (target - currArea) / combinedWidth`
    - Return `prevY + deltaY`
  - **Line 44**: Update accumulated area
  - **Line 45**: Update `combinedWidth` for the next interval (add/subtract based on event)
  - **Line 46**: Update previous y-coordinate

**Key Formula:**
```
deltaY = (target - currArea) / combinedWidth
answer = prevY + deltaY
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
