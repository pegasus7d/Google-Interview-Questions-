# Shortest Path to Favorite City

## Problem Statement

Given:
- `N` cities numbered from 0 to N-1
- `M` roads, each connecting a pair of cities with a travel time
- A list of favorite cities `L`
- A source city `S`

Find the **favorite city** that can be reached from the source city in the **minimum time**. If multiple favorite cities have the same minimum time, return any one of them.

**Input Format:**
- `N`: Number of cities
- `M`: Number of roads
- `roads`: List of `[u, v, time]` representing a bidirectional road between cities `u` and `v` with travel time `time`
- `L`: List of favorite city indices
- `S`: Source city index

**Output:** The favorite city that can be reached in minimum time, or -1 if no favorite city is reachable.

## Examples

### Example 1
```
Input:
N = 5, M = 6
roads = [[0, 1, 2], [0, 2, 4], [1, 2, 1], [1, 3, 7], [2, 3, 3], [3, 4, 1]]
L = [2, 4]
S = 0

Graph:
    0 --2-- 1 --7-- 3 --1-- 4
    |       |       |
    4       1       3
    |       |       |
    2 --1-- 2       

Shortest distances from S=0:
- City 0: 0
- City 1: 2
- City 2: 3 (via 0->1->2)
- City 3: 6 (via 0->1->2->3)
- City 4: 7 (via 0->1->2->3->4)

Favorite cities: [2, 4]
- City 2: distance = 3
- City 4: distance = 7

Output: 2 (minimum distance among favorites)
```

### Example 2
```
Input:
N = 4, M = 4
roads = [[0, 1, 1], [1, 2, 2], [2, 3, 3], [0, 3, 10]]
L = [3]
S = 0

Shortest distances from S=0:
- City 0: 0
- City 1: 1
- City 2: 3 (via 0->1->2)
- City 3: 6 (via 0->1->2->3, better than direct 10)

Output: 3
```

### Example 3
```
Input:
N = 3, M = 2
roads = [[0, 1, 5], [1, 2, 3]]
L = [2]
S = 0

Shortest distances from S=0:
- City 0: 0
- City 1: 5
- City 2: 8 (via 0->1->2)

Output: 2
```

## Constraints
- `1 <= N <= 10^5`
- `1 <= M <= 10^5`
- `1 <= time <= 10^4`
- `0 <= S < N`
- `1 <= |L| <= N`
- All cities in `L` are valid (0 <= city < N)
- Graph may be disconnected

## Approach

### Step 1: Understanding the Problem
We need to find the shortest path from source `S` to any favorite city in `L`. This is a classic **single-source shortest path** problem, which can be solved using **Dijkstra's algorithm**.

### Step 2: Identifying the Strategy
Dijkstra's algorithm is suitable because:
- All edge weights are **non-negative** (travel times are positive)
- We need shortest paths from a single source to multiple destinations
- The algorithm efficiently finds shortest paths to all reachable nodes

**Key Insight:** Run Dijkstra from source `S`, then find the favorite city with minimum distance.

### Step 3: Algorithm Design
1. **Build Graph**: Create adjacency list representation
2. **Initialize**: 
   - Distance array `dist[]` initialized to infinity
   - Priority queue (min-heap) with `{distance, node}`
   - Set `dist[S] = 0` and push `{0, S}`
3. **Dijkstra's Algorithm**:
   - Extract node with minimum distance from priority queue
   - Skip if distance is stale (already processed with better distance)
   - Relax all neighbors: update distance if shorter path found
4. **Find Answer**: After Dijkstra completes, find favorite city with minimum distance

### Step 4: Edge Cases
- No favorite city is reachable: Return -1
- Source is a favorite city: Return source if distance is 0
- Graph is disconnected: Some favorite cities may be unreachable
- Multiple favorite cities with same minimum distance: Return any one

## Solution

### Code Implementation

```cpp
#include <vector>
#include <queue>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
public:
    /**
     * Find favorite city reachable from source in minimum time
     * 
     * @param N Number of cities
     * @param roads Vector of [u, v, time] representing bidirectional roads
     * @param L List of favorite city indices
     * @param S Source city index
     * @return Favorite city with minimum distance, or -1 if unreachable
     */
    int findFavoriteCity(int N, vector<vector<int>>& roads, vector<int>& L, int S) {
        // Build adjacency list
        vector<vector<pair<int, int>>> adj(N);
        for (auto& road : roads) {
            int u = road[0], v = road[1], time = road[2];
            adj[u].push_back({v, time});
            adj[v].push_back({u, time});  // Bidirectional
        }
        
        // Initialize distance array
        vector<int> dist(N, INT_MAX);
        dist[S] = 0;
        
        // Priority queue: {distance, node}
        // Using greater<> to make it a min-heap
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
        pq.push({0, S});
        
        // Dijkstra's algorithm
        while (!pq.empty()) {
            auto [d, u] = pq.top();
            pq.pop();
            
            // Skip if this is a stale entry (we've already found a better path)
            // Better way to write: if (d != dist[u]) continue;
            if (d > dist[u]) continue;
            
            // Relax all neighbors
            for (auto& [v, w] : adj[u]) {
                int newDist = d + w;
                if (newDist < dist[v]) {
                    dist[v] = newDist;
                    pq.push({dist[v], v});
                }
            }
        }
        
        // Find favorite city with minimum distance
        int minDist = INT_MAX;
        int result = -1;
        
        for (int city : L) {
            if (dist[city] < minDist) {
                minDist = dist[city];
                result = city;
            }
        }
        
        // Return -1 if no favorite city is reachable
        return (minDist == INT_MAX) ? -1 : result;
    }
};
```

### Explanation
- **Line 18-24**: Build bidirectional graph using adjacency list
- **Line 26-28**: Initialize distance array with infinity, set source distance to 0
- **Line 30-32**: Create min-heap priority queue and push source node
- **Line 34-48**: Dijkstra's algorithm:
  - **Line 37-38**: Extract node with minimum distance
  - **Line 40-41**: **Skip stale entries**: If `d > dist[u]`, this entry was added with an old (larger) distance. We've already processed this node with a better distance, so skip it. Better way: `if (d != dist[u]) continue;` - this checks if the distance in queue matches current best distance.
  - **Line 43-48**: Relax neighbors: if we found a shorter path, update distance and push to queue
- **Line 50-60**: Find favorite city with minimum distance
- **Line 62**: Return -1 if no favorite city is reachable

**Why `d != dist[u]` is better than `d > dist[u]`:**
- `d > dist[u]`: Only checks if distance is larger (stale)
- `d != dist[u]`: Checks if distance doesn't match (more precise, handles edge cases better)
- Both work, but `d != dist[u]` is more explicit about checking for stale entries

## Complexity Analysis

### Time Complexity
- **Building graph**: O(M) - process each road once
- **Dijkstra's algorithm**: 
  - Each node is extracted from priority queue at most once: O(V log V)
  - Each edge is relaxed at most once: O(E log V)
  - **Total**: O((V + E) log V)
- **Finding minimum favorite city**: O(|L|) = O(V) in worst case
- **Overall**: O((V + E) log V)

### Space Complexity
- **Adjacency list**: O(V + E)
- **Distance array**: O(V)
- **Priority queue**: O(V) in worst case
- **Overall**: O(V + E)

### Sparse vs Dense Graphs

**Sparse Graph (E ≈ V):**
- Time: O(V log V) - dominated by node extractions
- Space: O(V)
- Example: Tree, grid graph

**Dense Graph (E ≈ V²):**
- Time: O(V² log V) - many edges to relax
- Space: O(V²)
- Example: Complete graph

**Note:** For dense graphs, if E = V², we can also express complexity as O(E log V).

## Key Insights
- **Dijkstra's Algorithm**: Optimal for single-source shortest paths with non-negative weights
- **Stale Entry Handling**: The `d != dist[u]` check is crucial - same node can be in queue multiple times with different distances
- **Priority Queue**: Min-heap ensures we always process the node with minimum distance first
- **Early Termination**: We can optimize by stopping when all favorite cities are processed (see alternative approach)
- **Bidirectional Edges**: Remember to add edges in both directions for undirected graphs

## Alternative Approaches

### Early Stoppage Optimization
If we want to stop as soon as we've processed all favorite cities:

```cpp
int findFavoriteCityWithEarlyStop(int N, vector<vector<int>>& roads, 
                                   vector<int>& L, int S) {
    // Build graph
    vector<vector<pair<int, int>>> adj(N);
    for (auto& road : roads) {
        int u = road[0], v = road[1], time = road[2];
        adj[u].push_back({v, time});
        adj[v].push_back({u, time});
    }
    
    // Mark favorite cities
    vector<bool> isFavorite(N, false);
    for (int city : L) {
        isFavorite[city] = true;
    }
    
    vector<int> dist(N, INT_MAX);
    dist[S] = 0;
    
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, S});
    
    int favoriteCount = 0;
    int minFavoriteDist = INT_MAX;
    int result = -1;
    
    while (!pq.empty() && favoriteCount < L.size()) {
        auto [d, u] = pq.top();
        pq.pop();
        
        if (d != dist[u]) continue;
        
        // If this is a favorite city, update result
        if (isFavorite[u]) {
            if (d < minFavoriteDist) {
                minFavoriteDist = d;
                result = u;
            }
            favoriteCount++;
        }
        
        // Relax neighbors
        for (auto& [v, w] : adj[u]) {
            int newDist = d + w;
            if (newDist < dist[v]) {
                dist[v] = newDist;
                pq.push({dist[v], v});
            }
        }
    }
    
    return result;
}
```

**Time Complexity:** Still O((V + E) log V) in worst case, but can be faster if favorite cities are close to source.

### Variation: Must Pass Through Fixed Vertex V

**Problem Extension:** Find the favorite city reachable from source `S` in minimum time, but the path **must pass through** a fixed vertex `V`.

**Solution Approach:**
1. Run Dijkstra from `S` to all nodes → get `dist1[]`
2. Run Dijkstra from `V` to all nodes → get `dist2[]`
3. For each favorite city `f` in `L`:
   - Total distance = `dist1[V] + dist2[f]` (path: S → V → f)
4. Return favorite city with minimum total distance

```cpp
// Helper function for Dijkstra's algorithm
vector<int> dijkstra(int N, vector<vector<pair<int, int>>>& adj, int source) {
    vector<int> dist(N, INT_MAX);
    dist[source] = 0;
    
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, source});
    
    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        
        if (d != dist[u]) continue;
        
        for (auto& [v, w] : adj[u]) {
            int newDist = d + w;
            if (newDist < dist[v]) {
                dist[v] = newDist;
                pq.push({dist[v], v});
            }
        }
    }
    
    return dist;
}

int findFavoriteCityViaVertex(int N, vector<vector<int>>& roads, 
                               vector<int>& L, int S, int V) {
    // Build graph
    vector<vector<pair<int, int>>> adj(N);
    for (auto& road : roads) {
        int u = road[0], v = road[1], time = road[2];
        adj[u].push_back({v, time});
        adj[v].push_back({u, time});
    }
    
    // Run Dijkstra from S and V
    vector<int> distFromS = dijkstra(N, adj, S);
    vector<int> distFromV = dijkstra(N, adj, V);
    
    // Check if V is reachable from S
    if (distFromS[V] == INT_MAX) {
        return -1;  // V is not reachable from S
    }
    
    // Find favorite city with minimum distance via V
    int minDist = INT_MAX;
    int result = -1;
    
    for (int city : L) {
        if (distFromV[city] == INT_MAX) continue;  // City not reachable from V
        
        int totalDist = distFromS[V] + distFromV[city];
        if (totalDist < minDist) {
            minDist = totalDist;
            result = city;
        }
    }
    
    return (minDist == INT_MAX) ? -1 : result;
}
```

**Time Complexity:** 
- Two Dijkstra runs: O((V + E) log V) each
- Finding minimum: O(|L|)
- **Total**: O((V + E) log V)

**Space Complexity:** O(V + E)

**Explanation:**
- First Dijkstra finds shortest paths from `S` to all nodes (including `V`)
- Second Dijkstra finds shortest paths from `V` to all nodes (including favorite cities)
- For any favorite city `f`, the shortest path `S → V → f` has distance `distFromS[V] + distFromV[f]`
- We choose the favorite city with minimum such distance

## Related Problems
- Network Delay Time
- Cheapest Flights Within K Stops
- Path With Minimum Effort
- Find the City With the Smallest Number of Neighbors at a Threshold Distance
- Shortest Path in Binary Matrix

