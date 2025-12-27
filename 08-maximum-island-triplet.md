# Maximum Island Triplet Sum

## Problem Statement

Given:
- `n` islands (numbered from 0 to n-1)
- Each island has a value
- Some islands are directly connected (undirected edges)

Select a group of **3 islands A, B, C** such that:
- They form a **directly connected path**: A is directly connected to B, and B is directly connected to C
- There is **no other island in between** (A-B-C is a simple path of length 2)
- The **sum of values** of these 3 islands is **maximum**

**Input Format:**
- `n`: Number of islands
- `values`: Array of island values
- `edges`: List of `[u, v]` representing connections between islands

**Output:** Maximum sum of values of three islands forming a path of length 2.

## Examples

### Example 1
```
Input:
n = 5
values = [10, 20, 30, 40, 50]
edges = [[0, 1], [1, 2], [2, 3], [1, 4]]

Graph:
    0(10) -- 1(20) -- 2(30) -- 3(40)
              |
              4(50)

Valid triplets (A-B-C paths):
- 0-1-2: sum = 10 + 20 + 30 = 60
- 0-1-4: sum = 10 + 20 + 50 = 80
- 1-2-3: sum = 20 + 30 + 40 = 90
- 2-1-4: sum = 30 + 20 + 50 = 100

Output: 100
```

### Example 2
```
Input:
n = 4
values = [5, 10, 15, 20]
edges = [[0, 1], [1, 2], [2, 3]]

Graph:
0(5) -- 1(10) -- 2(15) -- 3(20)

Valid triplets:
- 0-1-2: sum = 5 + 10 + 15 = 30
- 1-2-3: sum = 10 + 15 + 20 = 45

Output: 45
```

### Example 3
```
Input:
n = 3
values = [1, 2, 3]
edges = [[0, 1], [1, 2]]

Graph:
0(1) -- 1(2) -- 2(3)

Valid triplet:
- 0-1-2: sum = 1 + 2 + 3 = 6

Output: 6
```

## Constraints
- `3 <= n <= 10^4`
- `1 <= values[i] <= 10^4`
- `1 <= edges.length <= 10^4`
- Graph is undirected and may be disconnected
- No self-loops or multiple edges

## Approach

### Step 1: Understanding the Problem
We need to find three islands A, B, C that form a **path of length 2**:
- A is directly connected to B
- B is directly connected to C
- A, B, C are distinct islands

**Key Insight:** For any path A-B-C, island B is the **middle island**. We can fix B and find the best A and C from its neighbors.

### Step 2: Identifying the Strategy
**Brute Force Approach:**
- For each island B (middle), try all pairs of neighbors (A, C)
- Time: O(n × d²) where d is maximum degree (can be O(n²) in worst case)

**Optimized Approach:**
- For each island B (middle), we only need the **top 2 neighbors by value**
- Why? Because we want to maximize `value[A] + value[B] + value[C]`
- If B has neighbors sorted by value, the maximum sum comes from the top 2
- Time: O(n × d) = O(n × m) where m is number of edges

### Step 3: Algorithm Design
1. **Build adjacency list** from edges
2. **For each island B** (potential middle):
   - Find its **top 2 neighbors** by value
   - If B has at least 2 neighbors:
     - Compute sum = `value[top1] + value[B] + value[top2]`
     - Update maximum
3. **Return** maximum sum found

### Step 4: Edge Cases
- Island with less than 2 neighbors: Cannot form a triplet, skip
- Disconnected graph: Process each component separately
- Multiple edges between same islands: Handled by adjacency list (no duplicates)
- All islands have same value: Return any valid triplet

## Solution

### Code Implementation

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
public:
    /**
     * Find maximum sum of three islands forming a path of length 2
     * 
     * @param n Number of islands
     * @param values Array of island values
     * @param edges List of [u, v] representing connections
     * @return Maximum sum of three islands forming A-B-C path
     */
    int maxIslandValueSum(int n, vector<int>& values, vector<vector<int>>& edges) {
        // Build adjacency list
        vector<vector<int>> adj(n);
        for (auto& e : edges) {
            int u = e[0];
            int v = e[1];
            adj[u].push_back(v);
            adj[v].push_back(u);
        }
        
        int maxSum = -1;
        
        // Fix each island as the middle island B
        for (int b = 0; b < n; b++) {
            // Find top 2 neighbors of B by value
            int first = -1;   // Neighbor with highest value
            int second = -1;  // Neighbor with second highest value
            
            for (int nei : adj[b]) {
                if (first == -1 || values[nei] > values[first]) {
                    // New highest value neighbor
                    second = first;
                    first = nei;
                } else if (second == -1 || values[nei] > values[second]) {
                    // New second highest value neighbor
                    second = nei;
                }
            }
            
            // If B has at least 2 neighbors, we can form a triplet
            if (first != -1 && second != -1) {
                int sum = values[first] + values[b] + values[second];
                maxSum = max(maxSum, sum);
            }
        }
        
        return maxSum;
    }
};
```

### Explanation
- **Line 18-23**: Build undirected graph using adjacency list
- **Line 25**: Initialize maximum sum to -1 (or INT_MIN)
- **Line 27-45**: For each island `b` (potential middle):
  - **Line 29-30**: Track top 2 neighbors by value
  - **Line 32-40**: Iterate through all neighbors of `b`:
    - **Line 33-36**: Update if we find a neighbor with higher value than current `first`
    - **Line 37-39**: Update if we find a neighbor with higher value than current `second`
  - **Line 42-45**: If `b` has at least 2 neighbors, compute sum and update maximum
- **Line 47**: Return maximum sum found

**Why Top 2 Neighbors?**
- For path A-B-C, we need two distinct neighbors of B
- To maximize `value[A] + value[B] + value[C]`, we want the two neighbors with highest values
- This gives us the maximum possible sum for any triplet centered at B

**Time Complexity:** O(n + m) where m is number of edges
- Build graph: O(m)
- Process each island: O(degree) per island
- Total: O(n + m)

## Complexity Analysis

### Time Complexity
- **O(n + m)**: 
  - Building adjacency list: O(m) where m is number of edges
  - For each island, we iterate through its neighbors: O(degree)
  - Sum of all degrees = 2m (each edge counted twice)
  - **Overall**: O(n + m)

### Space Complexity
- **O(n + m)**: 
  - Adjacency list: O(n + m)
  - Other variables: O(1)
  - **Overall**: O(n + m)

## Key Insights
- **Middle Island Strategy**: Fix the middle island B, then find best neighbors A and C
- **Greedy Selection**: For each middle island, we only need the top 2 neighbors by value
- **No Need for Full BFS**: We don't need to explore 3 levels - just check direct neighbors
- **Path of Length 2**: A-B-C means exactly 2 edges, no intermediate nodes
- **Undirected Graph**: Each edge is bidirectional, so we check both directions naturally

## Alternative Approaches

### Brute Force (Check All Pairs)

```cpp
int maxIslandValueSumBruteForce(int n, vector<int>& values, 
                                 vector<vector<int>>& edges) {
    vector<vector<int>> adj(n);
    for (auto& e : edges) {
        adj[e[0]].push_back(e[1]);
        adj[e[1]].push_back(e[0]);
    }
    
    int maxSum = -1;
    
    // For each middle island B
    for (int b = 0; b < n; b++) {
        // Try all pairs of neighbors
        for (int i = 0; i < adj[b].size(); i++) {
            for (int j = i + 1; j < adj[b].size(); j++) {
                int a = adj[b][i];
                int c = adj[b][j];
                int sum = values[a] + values[b] + values[c];
                maxSum = max(maxSum, sum);
            }
        }
    }
    
    return maxSum;
}
```

**Time Complexity:** O(n × d²) where d is maximum degree  
**Space Complexity:** O(n + m)

**When to use:** When maximum degree is small (sparse graph)

### Using Priority Queue (For Dense Graphs)

```cpp
int maxIslandValueSumWithPQ(int n, vector<int>& values, 
                            vector<vector<int>>& edges) {
    vector<vector<int>> adj(n);
    for (auto& e : edges) {
        adj[e[0]].push_back(e[1]);
        adj[e[1]].push_back(e[0]);
    }
    
    int maxSum = -1;
    
    for (int b = 0; b < n; b++) {
        if (adj[b].size() < 2) continue;
        
        // Use priority queue to get top 2 neighbors
        priority_queue<pair<int, int>> pq;  // {value, neighbor_index}
        
        for (int nei : adj[b]) {
            pq.push({values[nei], nei});
        }
        
        auto [val1, a] = pq.top(); pq.pop();
        auto [val2, c] = pq.top();
        
        int sum = val1 + values[b] + val2;
        maxSum = max(maxSum, sum);
    }
    
    return maxSum;
}
```

**Time Complexity:** O(n × d log d)  
**Space Complexity:** O(n + m)

**When to use:** When we need top k neighbors (k > 2)

## Related Problems
- Maximum Path Sum in Binary Tree
- Longest Path in a Tree
- Maximum Sum of Three Non-Overlapping Subarrays
- Maximum Product of Three Numbers

