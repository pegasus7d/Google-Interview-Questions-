# Question-Volunteer Matching (Maximum Bipartite Matching)

## Problem Statement

Given a list of **questions** and a list of **volunteers**, assign questions to volunteers such that:

1. Each question can be assigned to **at most one volunteer**
2. Each volunteer can take **at most one question**
3. A volunteer can only take a question if they share **at least one common tag**
4. **Goal**: Maximize the number of questions assigned to volunteers

**Data Structures:**
```cpp
// Question: has id and tags
struct Question {
    int id;
    vector<string> tags;
};

// Volunteer: has id, tags, and name
struct Volunteer {
    string id;
    vector<string> tags;
    string name;
};
```

**Input Format:**
- `questionTags`: Vector of vectors, where `questionTags[i]` contains tags for question `i`
- `volunteerTags`: Vector of vectors, where `volunteerTags[i]` contains tags for volunteer `i`
- `volunteerNames`: Vector of volunteer names

**Output:** Maximum number of assignments and the actual assignments (question → volunteer).

## Examples

### Example 1
```
Input:
Questions:
  Q1: ["MAC", "VSCODE"]
  Q2: ["PY", "AI"]
  Q3: ["JAVA", "OS"]
  Q4: ["PY", "NW"]

Volunteers:
  A: ["PY", "NW"]
  B: ["AI"]
  C: ["JAVA", "NW"]
  D: ["JAVA", "NW"]

Bipartite Graph:
  A → Q4 (PY match)
  B → Q2 (AI match)
  C → Q3 (JAVA match)
  D → Q3 (JAVA match) [but Q3 already taken]

Maximum Matching:
  A → Q4
  B → Q2
  C → Q3
  Q1 → unassigned (no matching tags)

Output:
  Total matches: 3
  Volunteer A assigned to Question 4
  Volunteer B assigned to Question 2
  Volunteer C assigned to Question 3
```

### Example 2
```
Input:
Questions:
  Q1: ["PY", "AI"]
  Q2: ["JAVA", "OS"]
  Q3: ["PY", "NW"]

Volunteers:
  A: ["PY", "AI"]
  B: ["JAVA"]
  C: ["PY"]

Bipartite Graph:
  A → Q1 (PY, AI match), Q3 (PY match)
  B → Q2 (JAVA match)
  C → Q1 (PY match), Q3 (PY match)

Maximum Matching:
  A → Q1
  B → Q2
  C → Q3

Output:
  Total matches: 3
  All questions assigned
```

### Example 3
```
Input:
Questions:
  Q1: ["C++", "OS"]
  Q2: ["PY", "AI"]

Volunteers:
  A: ["JAVA"]
  B: ["PY"]

Bipartite Graph:
  A → (no matches)
  B → Q2 (PY match)

Maximum Matching:
  B → Q2
  Q1 → unassigned

Output:
  Total matches: 1
  Volunteer B assigned to Question 2
```

## Constraints
- `1 <= questions.length <= 100`
- `1 <= volunteers.length <= 100`
- `1 <= tags per question/volunteer <= 10`
- Tags are case-sensitive strings
- Each volunteer name is unique
- Each question id is unique

## Approach

### Step 1: Understanding the Problem
This is a **maximum bipartite matching** problem:
- **Left side**: Volunteers
- **Right side**: Questions
- **Edge exists**: If volunteer and question share at least one common tag
- **Goal**: Find maximum matching (maximum number of edges with no shared vertices)

**Key Constraints:**
- One-to-one matching (each volunteer → at most one question)
- Matching must respect tag compatibility
- Maximize number of assignments

### Step 2: Identifying the Strategy
**DFS-based Augmenting Path Algorithm:**
- Classic algorithm for maximum bipartite matching
- For each volunteer, try to find an augmenting path
- If question is free, assign directly
- If question is taken, try to reassign the current volunteer of that question

**Why DFS?**
- Need to explore paths to find augmenting paths
- Backtracking allows reassignment of existing matches
- Efficient for bipartite graphs

**Algorithm:**
1. Build bipartite graph (volunteer → question edges based on tag matching)
2. For each volunteer, run DFS to find augmenting path
3. If path found, update matching
4. Return maximum number of matches

### Step 3: Algorithm Design
1. **Build bipartite graph**: Create adjacency matrix `grid[v][q] = 1` if volunteer `v` can take question `q`
2. **Initialize matching**: `match[q] = -1` means question `q` is unassigned
3. **For each volunteer**:
   - Run DFS to find augmenting path
   - If found, update matching
4. **DFS function**:
   - For each question volunteer can take:
     - If question is free → assign directly
     - If question is taken → recursively try to reassign old volunteer
     - Use visited array to avoid cycles in current DFS

### Step 4: Edge Cases
- No matching tags: Some questions/volunteers may remain unassigned
- All volunteers can take same question: Only one gets assigned
- Volunteer can take multiple questions: DFS finds best assignment
- More volunteers than questions: Some volunteers remain unassigned
- More questions than volunteers: Some questions remain unassigned

## Solution

### Code Implementation

```cpp
#include <vector>
#include <string>
#include <unordered_set>
using namespace std;

class Solution {
private:
    /**
     * DFS to find augmenting path for a volunteer
     * 
     * @param volunteer Current volunteer index
     * @param grid Bipartite graph: grid[v][q] = 1 if volunteer v can take question q
     * @param match match[question] = volunteer assigned to that question (-1 if free)
     * @param visited visited[question] = true if question visited in current DFS
     * @return true if augmenting path found, false otherwise
     */
    bool dfs(
        int volunteer,
        vector<vector<int>>& grid,
        vector<int>& match,
        vector<bool>& visited
    ) {
        int Q = grid[0].size();
        
        // Try all questions this volunteer can take
        for (int question = 0; question < Q; question++) {
            // If no edge or already tried this question in this DFS
            if (grid[volunteer][question] == 0 || visited[question]) {
                continue;
            }
            
            visited[question] = true;
            
            // CASE 1: Question is free → assign directly
            if (match[question] == -1) {
                match[question] = volunteer;
                return true;  // Augmenting path found
            }
            
            // CASE 2: Question is taken → try to reassign old volunteer
            int oldVolunteer = match[question];
            if (dfs(oldVolunteer, grid, match, visited)) {
                // Old volunteer found alternative assignment
                match[question] = volunteer;  // Reassign to current volunteer
                return true;  // Augmenting path found
            }
        }
        
        // No augmenting path found for this volunteer
        return false;
    }
    
public:
    /**
     * Find maximum matching between volunteers and questions
     * 
     * @param questionTags Tags for each question
     * @param volunteerTags Tags for each volunteer
     * @param volunteerNames Names of volunteers
     * @return Maximum number of matches
     */
    int maximumMatching(
        vector<vector<string>>& questionTags,
        vector<vector<string>>& volunteerTags,
        vector<string>& volunteerNames
    ) {
        int V = volunteerTags.size();
        int Q = questionTags.size();
        
        // -------- BUILD BIPARTITE GRAPH --------
        // grid[v][q] = 1 means volunteer v can take question q
        vector<vector<int>> grid(V, vector<int>(Q, 0));
        
        for (int v = 0; v < V; v++) {
            // Convert volunteer tags to set for fast lookup
            unordered_set<string> vtags(
                volunteerTags[v].begin(), 
                volunteerTags[v].end()
            );
            
            for (int q = 0; q < Q; q++) {
                // Check if volunteer has any tag matching question tags
                for (auto& tag : questionTags[q]) {
                    if (vtags.count(tag)) {
                        grid[v][q] = 1;  // Edge exists
                        break;  // One match is enough
                    }
                }
            }
        }
        
        // -------- MATCHING STRUCTURE --------
        // match[question] = volunteer assigned to that question (-1 if free)
        vector<int> match(Q, -1);
        
        int matches = 0;
        
        // Try to find an augmenting path for each volunteer
        for (int v = 0; v < V; v++) {
            vector<bool> visited(Q, false);
            if (dfs(v, grid, match, visited)) {
                matches++;
            }
        }
        
        return matches;
    }
    
    /**
     * Get detailed assignment mapping
     */
    map<int, string> getAssignments(
        vector<vector<string>>& questionTags,
        vector<vector<string>>& volunteerTags,
        vector<string>& volunteerNames
    ) {
        int V = volunteerTags.size();
        int Q = questionTags.size();
        
        // Build graph and find matching (same as above)
        vector<vector<int>> grid(V, vector<int>(Q, 0));
        
        for (int v = 0; v < V; v++) {
            unordered_set<string> vtags(
                volunteerTags[v].begin(), 
                volunteerTags[v].end()
            );
            
            for (int q = 0; q < Q; q++) {
                for (auto& tag : questionTags[q]) {
                    if (vtags.count(tag)) {
                        grid[v][q] = 1;
                        break;
                    }
                }
            }
        }
        
        vector<int> match(Q, -1);
        
        for (int v = 0; v < V; v++) {
            vector<bool> visited(Q, false);
            dfs(v, grid, match, visited);
        }
        
        // Build result mapping
        map<int, string> assignments;
        for (int q = 0; q < Q; q++) {
            if (match[q] != -1) {
                assignments[q] = volunteerNames[match[q]];
            }
        }
        
        return assignments;
    }
};
```

### Explanation
- **Line 20-60**: `dfs` function (augmenting path search):
  - **Line 30-33**: Try all questions volunteer can take
  - **Line 35-37**: Skip if no edge or already visited in current DFS
  - **Line 39**: Mark question as visited (prevents cycles)
  - **Line 41-45**: **Case 1 - Question is free**:
    - Assign directly
    - Return true (augmenting path found)
  - **Line 47-53**: **Case 2 - Question is taken**:
    - Get current volunteer assigned to question
    - Recursively try to find alternative for old volunteer
    - If successful, reassign question to current volunteer
    - Return true (augmenting path found)
  - **Line 56**: Return false if no augmenting path found
  
- **Line 63-120**: `maximumMatching` function:
  - **Line 70-88**: **Build bipartite graph**:
    - Create adjacency matrix `grid[V][Q]`
    - For each volunteer, check tag matches with each question
    - Set `grid[v][q] = 1` if at least one tag matches
  - **Line 90-92**: Initialize matching array (all questions free)
  - **Line 94-100**: **Find maximum matching**:
    - For each volunteer, run DFS to find augmenting path
    - If found, increment match count
  - **Line 102**: Return total number of matches

**Why This Works:**
- **Augmenting Path**: A path that alternates between unmatched and matched edges
- **DFS finds augmenting paths**: Allows reassignment of existing matches
- **Maximum matching**: Algorithm guarantees maximum number of matches
- **Visited array**: Prevents infinite loops in DFS (reset for each volunteer)

**Example Walkthrough:**
```
Volunteers: A["PY","NW"], B["AI"], C["JAVA","NW"]
Questions: Q1["MAC"], Q2["PY","AI"], Q3["JAVA"], Q4["PY","NW"]

Graph:
  A → Q2, Q4
  B → Q2
  C → Q3

Step 1: Volunteer A
  Try Q2: Free → Assign A→Q2, matches=1

Step 2: Volunteer B
  Try Q2: Taken by A
    Try to reassign A: A can take Q4 (free) → Reassign A→Q4
    Assign B→Q2, matches=2

Step 3: Volunteer C
  Try Q3: Free → Assign C→Q3, matches=3

Result: 3 matches (A→Q4, B→Q2, C→Q3)
```

## Complexity Analysis

### Time Complexity
- **O(V × Q × T)**: 
  - `V` = number of volunteers
  - `Q` = number of questions
  - `T` = average number of tags
  - Building graph: O(V × Q × T)
  - DFS for each volunteer: O(V × Q) in worst case
  - **Overall**: O(V × Q × T + V × Q) = O(V × Q × T)

### Space Complexity
- **O(V × Q)**: 
  - Graph matrix: O(V × Q)
  - Matching array: O(Q)
  - Visited array: O(Q)
  - **Overall**: O(V × Q)

## Key Insights
- **Bipartite Matching**: Classic graph problem with efficient algorithms
- **Augmenting Path**: Key concept - path that increases matching size
- **DFS for Reassignment**: Allows finding alternative assignments for existing matches
- **Visited Array**: Prevents cycles in DFS (reset for each volunteer)
- **Greedy Assignment**: Try to assign each volunteer, backtrack if needed
- **Maximum Matching**: Algorithm guarantees optimal solution
- **Tag Matching**: Build graph based on tag intersection (at least one common tag)

## Alternative Approaches

### Hopcroft-Karp Algorithm (Faster for Dense Graphs)

```cpp
// O(√V × E) algorithm for bipartite matching
// More complex but faster for large dense graphs
// Uses BFS to find multiple augmenting paths simultaneously
```

**Time Complexity:** O(√V × E) where E = number of edges  
**Space Complexity:** O(V + Q + E)

**When to Use:**
- Very large graphs
- Dense bipartite graphs
- Performance-critical applications

### Greedy Matching (Suboptimal)

```cpp
int greedyMatching(vector<vector<int>>& grid, int V, int Q) {
    vector<int> match(Q, -1);
    int matches = 0;
    
    for (int v = 0; v < V; v++) {
        for (int q = 0; q < Q; q++) {
            if (grid[v][q] == 1 && match[q] == -1) {
                match[q] = v;
                matches++;
                break;  // Take first available
            }
        }
    }
    
    return matches;
}
```

**Time Complexity:** O(V × Q)  
**Space Complexity:** O(Q)

**Why Not Optimal:**
- Doesn't consider reassignment
- May miss better matchings
- Example: Greedy might assign A→Q2, but B→Q2, A→Q4 is better

## Related Problems
- Maximum Bipartite Matching
- Assign Cookies
- Course Schedule II
- Maximum Flow (more general problem)

