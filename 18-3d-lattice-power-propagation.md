# 3D Lattice Power Propagation

## Problem Statement

Given a **3D lattice graph** (like a cube) where each node is either:
- **Torch node**: Has power value **16** (power source)
- **Wire node**: Has power value **0** (initially no power)

**Power Propagation Rules:**
1. If a torch node (power 16) is connected to a wire node (power 0), the wire node receives power **15** (1 lost during transmission)
2. If a wire node with power `p` is connected to another wire node, that node receives power `p - 1`
3. If a node can receive power from multiple paths, it keeps the **maximum power** it can receive
4. Power propagation stops when power becomes ≤ 1

**Example:**
```
16 -> 0 -> 0 becomes: 16 -> 15 -> 14

If there's another torch ahead:
16 -> 0 -> 0 -> 16 becomes: 16 -> 15 -> 15 <- 16
```

**Graph Structure:**
- 3D lattice (cube-like structure)
- Each node has **6 neighbors** (left, right, up, down, front, back)
- Nodes are connected in 3D space

**Input Format:**
- `grid`: 3D vector `vector<vector<vector<int>>>` representing the lattice
  - `grid[x][y][z] == 16`: Torch node
  - `grid[x][y][z] == 0`: Wire node

**Output:** Modify the grid in-place to show final power values after propagation.

## Examples

### Example 1
```
Input (2x2x2 grid):
Initial:
[0, 0]    [0, 0]
[0, 16]   [0, 0]

After propagation:
[14, 15]  [15, 16]
[15, 16]  [14, 15]

Explanation:
- Torch at (1,0,0) with power 16 propagates
- Neighbors receive power 15, then 14, etc.
```

### Example 2
```
Input (1x3x1 grid):
Initial: [16, 0, 0]

After propagation:
[16, 15, 14]

Explanation:
- Torch at position 0 propagates power
- Position 1: 16 - 1 = 15
- Position 2: 15 - 1 = 14
```

### Example 3
```
Input (1x4x1 grid):
Initial: [16, 0, 0, 16]

After propagation:
[16, 15, 15, 16]

Explanation:
- Two torches at positions 0 and 3
- Position 1: max(15 from left, 15 from right) = 15
- Position 2: max(14 from left, 15 from right) = 15
```

## Constraints
- `1 <= X, Y, Z <= 100` (grid dimensions)
- Grid contains only values 0 (wire) or 16 (torch)
- Power values are integers
- Each node has exactly 6 neighbors (in 3D space)

## Approach

### Step 1: Understanding the Problem
This is a **multi-source BFS propagation problem** in 3D space.

**Key Observations:**
- Power spreads from torch nodes (sources) to neighboring nodes
- Each transmission step reduces power by 1
- If a node can receive power from multiple paths, it keeps the **maximum power**
- Power propagation stops when power ≤ 1
- All torch nodes propagate simultaneously

**Modeling:**
- 3D grid: `grid[x][y][z]`
- Each node has 6 neighbors: (±x, ±y, ±z directions)
- Torch nodes (value 16) are starting points
- Wire nodes (value 0) receive and propagate power

### Step 2: Identifying the Strategy
**Multi-Source BFS:**
- All torch nodes (power == 16) act as starting points
- Power spreads outward level by level (distance-based)
- Each edge has the same "cost" (power - 1)
- Use BFS to ensure shortest path (maximum power)

**Why BFS?**
- BFS processes nodes in order of distance from sources
- Ensures we always propagate maximum possible power
- Natural level-by-level propagation

**Update Condition:**
- Only update a node if it receives **stronger power** than it currently has
- This avoids need for visited array
- Guarantees strongest signal wins

### Step 3: Algorithm Design
1. **Multi-source initialization**: Add all torch nodes (power 16) to queue
2. **BFS propagation**:
   - Pop node from queue
   - If power ≤ 1, skip (can't propagate further)
   - For each of 6 neighbors:
     - Calculate `nextPower = currentPower - 1`
     - If `nextPower > grid[neighbor]`, update and push to queue
3. **Termination**: When queue is empty (no more updates possible)

**Why No Visited Array?**
- A node may need to be revisited if stronger power reaches it later
- Value-based pruning: only update when `nextPower > currentPower`
- Since power is bounded (≤ 16), process always terminates

### Step 4: Edge Cases
- No torch nodes: Grid remains unchanged
- All torch nodes: All nodes get power 16
- Isolated torch: Power propagates only to connected components
- Multiple torches: Nodes receive maximum power from nearest torch
- Boundary nodes: Check bounds before accessing neighbors

## Solution

### Code Implementation

```cpp
#include <vector>
#include <queue>
using namespace std;

// Directions in 3D: left, right, up, down, front, back
int dx[6] = {1, -1, 0, 0, 0, 0};
int dy[6] = {0, 0, 1, -1, 0, 0};
int dz[6] = {0, 0, 0, 0, 1, -1};

struct Cell {
    int x, y, z;
};

class Solution {
public:
    /**
     * Propagate power from torch nodes to wire nodes in 3D lattice
     * 
     * @param grid 3D grid where 16 = torch, 0 = wire
     * Modifies grid in-place with final power values
     */
    void propagatePower(vector<vector<vector<int>>>& grid) {
        int X = grid.size();
        int Y = grid[0].size();
        int Z = grid[0][0].size();
        
        queue<Cell> q;
        
        // -------- MULTI-SOURCE INITIALIZATION --------
        // All torch nodes (power 16) are BFS sources
        for (int x = 0; x < X; x++) {
            for (int y = 0; y < Y; y++) {
                for (int z = 0; z < Z; z++) {
                    if (grid[x][y][z] == 16) {
                        q.push({x, y, z});
                    }
                }
            }
        }
        
        // -------- BFS PROPAGATION --------
        while (!q.empty()) {
            Cell cur = q.front();
            q.pop();
            
            int curPower = grid[cur.x][cur.y][cur.z];
            
            // No propagation possible if power is 1 or less
            if (curPower <= 1) {
                continue;
            }
            
            // Check all 6 neighbors in 3D space
            for (int d = 0; d < 6; d++) {
                int nx = cur.x + dx[d];
                int ny = cur.y + dy[d];
                int nz = cur.z + dz[d];
                
                // Bounds check
                if (nx < 0 || ny < 0 || nz < 0 ||
                    nx >= X || ny >= Y || nz >= Z) {
                    continue;
                }
                
                int nextPower = curPower - 1;
                
                // Only update if stronger power is possible
                // This ensures maximum power wins and avoids infinite loops
                if (nextPower > grid[nx][ny][nz]) {
                    grid[nx][ny][nz] = nextPower;
                    q.push({nx, ny, nz});
                }
            }
        }
    }
};
```

### Explanation
- **Line 8-10**: Define 6 directions in 3D space:
  - `dx`: ±1 in x-direction (left/right)
  - `dy`: ±1 in y-direction (up/down)
  - `dz`: ±1 in z-direction (front/back)
  
- **Line 12-15**: Cell structure to represent 3D coordinates

- **Line 25-35**: **Multi-source initialization**:
  - Iterate through entire grid
  - Find all torch nodes (value == 16)
  - Add them to BFS queue as starting points
  
- **Line 37-62**: **BFS propagation**:
  - **Line 40-44**: Pop current cell and get its power
  - **Line 46-49**: Skip if power ≤ 1 (can't propagate further)
  - **Line 51-61**: Check all 6 neighbors:
    - **Line 52-54**: Calculate neighbor coordinates
    - **Line 56-60**: Bounds check
    - **Line 62**: Calculate power for neighbor: `currentPower - 1`
    - **Line 64-68**: **Key update condition**:
      - Only update if `nextPower > grid[neighbor]`
      - This ensures maximum power wins
      - Prevents infinite loops (power always decreases)
      - Update grid and push neighbor to queue

**Why This Works:**
- **Multi-source BFS**: All torches propagate simultaneously
- **Value-based pruning**: Only update when receiving stronger power
- **Natural termination**: Power decreases, so process always ends
- **Maximum power**: BFS ensures shortest path (highest power) reaches first

**Example Walkthrough:**
```
Grid: [16, 0, 0, 16] (1x4x1)

Step 1: Initialize queue with positions 0 and 3 (both have power 16)
Step 2: Process position 0:
  - Neighbor at position 1: 16 - 1 = 15 > 0, update to 15, push
Step 3: Process position 3:
  - Neighbor at position 2: 16 - 1 = 15 > 0, update to 15, push
Step 4: Process position 1 (power 15):
  - Neighbor at position 2: 15 - 1 = 14, but 14 < 15, skip
  - Neighbor at position 0: 15 - 1 = 14 < 16, skip
Step 5: Process position 2 (power 15):
  - Neighbor at position 1: 15 - 1 = 14 < 15, skip
  - Neighbor at position 3: 15 - 1 = 14 < 16, skip

Final: [16, 15, 15, 16]
```

## Complexity Analysis

### Time Complexity
- **O(X × Y × Z)**: 
  - Each cell can be updated at most **16 times** (power values 16 down to 1)
  - In practice, each cell is processed a constant number of times
  - BFS visits each cell at most once per power level
  - **Overall**: O(X × Y × Z) with constant factor ≤ 16

### Space Complexity
- **O(X × Y × Z)**: 
  - Grid storage: O(X × Y × Z)
  - Queue: O(X × Y × Z) in worst case (all cells in queue)
  - **Overall**: O(X × Y × Z)

## Key Insights
- **Multi-Source BFS**: All torch nodes start propagation simultaneously
- **Value-Based Pruning**: No visited array needed - update only when receiving stronger power
- **Maximum Power Wins**: If multiple paths exist, node keeps maximum power
- **Natural Termination**: Power decreases by 1 each step, so process always ends
- **3D Neighbors**: Each node has exactly 6 neighbors (±x, ±y, ±z)
- **BFS Guarantees**: Shortest path (highest power) reaches first
- **Update Condition**: `nextPower > currentPower` ensures optimal propagation

## Alternative Approaches

### DFS with Memoization (Less Efficient)

```cpp
void propagatePowerDFS(vector<vector<vector<int>>>& grid, int x, int y, int z, int power) {
    if (power <= 1) return;
    if (x < 0 || y < 0 || z < 0 || 
        x >= grid.size() || y >= grid[0].size() || z >= grid[0][0].size()) {
        return;
    }
    
    if (power <= grid[x][y][z]) return; // Already has better power
    
    grid[x][y][z] = power;
    
    for (int d = 0; d < 6; d++) {
        propagatePowerDFS(grid, x + dx[d], y + dy[d], z + dz[d], power - 1);
    }
}

void propagatePower(vector<vector<vector<int>>>& grid) {
    int X = grid.size(), Y = grid[0].size(), Z = grid[0][0].size();
    
    for (int x = 0; x < X; x++) {
        for (int y = 0; y < Y; y++) {
            for (int z = 0; z < Z; z++) {
                if (grid[x][y][z] == 16) {
                    propagatePowerDFS(grid, x, y, z, 16);
                }
            }
        }
    }
}
```

**Time Complexity:** O(X × Y × Z × 16) - less efficient due to recursion overhead  
**Space Complexity:** O(X × Y × Z) - recursion stack

**Why BFS is Better:**
- More efficient (no recursion overhead)
- Natural level-by-level processing
- Easier to understand and implement

### Dijkstra's Algorithm (Overkill)

```cpp
// Using priority queue, but unnecessary since all edges have same weight
// BFS is simpler and more efficient for this problem
```

**Time Complexity:** O(X × Y × Z × log(X×Y×Z))  
**Space Complexity:** O(X × Y × Z)

**Why Not Needed:**
- All edges have same weight (power - 1)
- BFS is simpler and more efficient

## Related Problems
- Rotting Oranges (multi-source BFS)
- Walls and Gates
- Shortest Path in Binary Matrix
- 01 Matrix (distance from nearest 0)

