# Step-By-Step Directions From Binary Tree Node to Another

## Problem Statement

You are given:
- The root of a binary tree with `n` nodes
- Each node is uniquely assigned a value from 1 to `n`
- An integer `startValue` representing the value of the start node `s`
- An integer `destValue` representing the value of the destination node `t`

Find the **shortest path** starting from node `s` and ending at node `t`. Generate step-by-step directions as a string consisting of only:
- **'L'**: Go from a node to its **left child**
- **'R'**: Go from a node to its **right child**
- **'U'**: Go from a node to its **parent**

Return the step-by-step directions of the shortest path from node `s` to node `t`.

## Examples

### Example 1
```
Input: 
root = [5,1,2,3,null,6,4]
startValue = 3
destValue = 6

Tree:
        5
       / \
      1   2
     /   / \
    3   6   4

Path: 3 → 1 → 5 → 2 → 6
Directions:
- 3 → 1: U (go to parent)
- 1 → 5: U (go to parent)
- 5 → 2: R (go to right child)
- 2 → 6: L (go to left child)

Output: "UURL"
```

### Example 2
```
Input:
root = [2,1]
startValue = 2
destValue = 1

Tree:
    2
   /
  1

Path: 2 → 1
Directions:
- 2 → 1: L (go to left child)

Output: "L"
```

### Example 3
```
Input:
root = [1,2,3,4,5,6,7]
startValue = 4
destValue = 7

Tree:
        1
       / \
      2   3
     / \ / \
    4  5 6  7

Path: 4 → 2 → 1 → 3 → 7
Directions:
- 4 → 2: U
- 2 → 1: U
- 1 → 3: R
- 3 → 7: R

Output: "UURR"
```

## Constraints
- `2 <= n <= 10^5`
- `1 <= Node.val <= n`
- All values in the tree are unique
- `1 <= startValue, destValue <= n`
- `startValue != destValue`

## Approach

### Step 1: Understanding the Problem
We need to find the shortest path from node `s` to node `t` in a binary tree. The path can go:
- **Downward**: From parent to child (L or R)
- **Upward**: From child to parent (U)

**Key Insight:** The shortest path from `s` to `t` always goes through their **Lowest Common Ancestor (LCA)**.

### Step 2: Identifying the Strategy
**Path Structure:**
1. From `s` to LCA: All moves are **'U'** (going upward to parent)
2. From LCA to `t`: Moves are **'L'** or **'R'** (going downward)

**Algorithm:**
1. Find the **LCA** of `startValue` and `destValue`
2. Find path from **LCA to startValue** (convert all to 'U')
3. Find path from **LCA to destValue** (keep as 'L'/'R')
4. Combine: `"U" × (length of path from LCA to start) + path from LCA to dest`

### Step 3: Algorithm Design
1. **LCA Function:**
   - If root is null or equals `p` or `q`, return root
   - Recursively find LCA in left and right subtrees
   - If both left and right return non-null, current root is LCA
   - Otherwise, return the non-null result

2. **Find Path Function:**
   - DFS from root to target
   - Build path string with 'L' or 'R' as we traverse
   - Backtrack if path doesn't lead to target

3. **Combine Paths:**
   - Path from LCA to start: Convert all to 'U'
   - Path from LCA to dest: Keep as 'L'/'R'
   - Concatenate: `"U" × len + destPath`

### Step 4: Edge Cases
- Start and dest are siblings: Path is "U" + "L" or "U" + "R"
- Start is ancestor of dest: Only need path from start to dest
- Dest is ancestor of start: Only need "U" moves
- Start and dest are same: Shouldn't happen per constraints

## Solution

### Code Implementation

```cpp
#include <string>
using namespace std;

// Definition for a binary tree node
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) 
        : val(x), left(left), right(right) {}
};

class Solution {
private:
    /**
     * Find Lowest Common Ancestor of two nodes
     * 
     * @param root Current node
     * @param p Value of first node
     * @param q Value of second node
     * @return LCA node, or nullptr if not found
     */
    TreeNode* lca(TreeNode* root, int p, int q) {
        // Base cases
        if (root == nullptr || root->val == p || root->val == q) {
            return root;
        }
        
        // Recursively find LCA in left and right subtrees
        TreeNode* left = lca(root->left, p, q);
        TreeNode* right = lca(root->right, p, q);
        
        // If both left and right return non-null, current root is LCA
        if (left != nullptr && right != nullptr) {
            return root;
        }
        
        // Otherwise, return the non-null result (or null if both null)
        return (left != nullptr) ? left : right;
    }
    
    /**
     * Find path from root to target node
     * Builds path string with 'L' or 'R' directions
     * 
     * @param root Current node
     * @param target Target node value
     * @param path Path string being built (modified by reference)
     * @return true if target found, false otherwise
     */
    bool findPath(TreeNode* root, int target, string& path) {
        if (root == nullptr) {
            return false;
        }
        
        // Target found
        if (root->val == target) {
            return true;
        }
        
        // Try left subtree
        path.push_back('L');
        if (findPath(root->left, target, path)) {
            return true;
        }
        path.pop_back();  // Backtrack
        
        // Try right subtree
        path.push_back('R');
        if (findPath(root->right, target, path)) {
            return true;
        }
        path.pop_back();  // Backtrack
        
        return false;
    }
    
public:
    /**
     * Get step-by-step directions from startValue to destValue
     * 
     * @param root Root of the binary tree
     * @param startValue Value of start node
     * @param destValue Value of destination node
     * @return String of directions ('L', 'R', 'U')
     */
    string getDirections(TreeNode* root, int startValue, int destValue) {
        // Step 1: Find Lowest Common Ancestor
        TreeNode* lcaNode = lca(root, startValue, destValue);
        
        // Step 2: Find path from LCA to startValue
        string pathToStart = "";
        findPath(lcaNode, startValue, pathToStart);
        
        // Step 3: Find path from LCA to destValue
        string pathToDest = "";
        findPath(lcaNode, destValue, pathToDest);
        
        // Step 4: Convert path to start into 'U' moves
        // All moves from LCA to start are upward (to parent)
        string result(pathToStart.length(), 'U');
        
        // Step 5: Append path to destination (L/R moves)
        result += pathToDest;
        
        return result;
    }
};
```

### Explanation
- **Line 30-47**: `lca` function:
  - **Line 33-35**: Base cases: if root is null or equals `p` or `q`, return root
  - **Line 37-38**: Recursively find LCA in left and right subtrees
  - **Line 40-43**: If both subtrees return non-null, current root is the LCA
  - **Line 45-46**: Return the non-null result (or null if both are null)
  
- **Line 49-73**: `findPath` function:
  - **Line 52-54**: Base case: return false if root is null
  - **Line 56-59**: If target found, return true
  - **Line 61-66**: Try left subtree:
    - Add 'L' to path
    - If found, return true
    - Otherwise, backtrack (remove 'L')
  - **Line 68-73**: Try right subtree:
    - Add 'R' to path
    - If found, return true
    - Otherwise, backtrack (remove 'R')
  
- **Line 76-95**: `getDirections` function:
  - **Line 79**: Find LCA of start and destination nodes
  - **Line 81-83**: Find path from LCA to startValue (stored in `pathToStart`)
  - **Line 85-87**: Find path from LCA to destValue (stored in `pathToDest`)
  - **Line 89-90**: Convert all moves from LCA to start into 'U' (upward moves)
  - **Line 92-93**: Append path from LCA to dest (downward moves: 'L' or 'R')
  - **Line 95**: Return combined result

**Why This Works:**
- The shortest path from `s` to `t` always goes through their LCA
- From `s` to LCA: We need to go upward (all 'U' moves)
- From LCA to `t`: We need to go downward (L/R moves)
- This gives us the shortest path

## Complexity Analysis

### Time Complexity
- **O(n)**: 
  - LCA function: O(n) - visits each node at most once
  - `findPath` (called twice): O(n) each - DFS traversal
  - **Overall**: O(n)

### Space Complexity
- **O(h)**: 
  - Recursion stack: O(h) where h is tree height
  - Path strings: O(h) in worst case
  - **Overall**: O(h)
  - **Worst case** (skewed tree): O(n)
  - **Best case** (balanced tree): O(log n)

## Key Insights
- **LCA is Key**: The shortest path between two nodes in a tree always goes through their LCA
- **Path Structure**: Path = (Upward from start to LCA) + (Downward from LCA to dest)
- **Upward Moves**: All moves from start to LCA are 'U' (going to parent)
- **Downward Moves**: Moves from LCA to dest are 'L' or 'R' (going to children)
- **Backtracking**: Use backtracking in `findPath` to build correct path string
- **Tree Properties**: Unique node values allow us to search by value instead of node reference

## Alternative Approaches

### Using Parent Map (Iterative)

```cpp
string getDirectionsIterative(TreeNode* root, int startValue, int destValue) {
    // Build parent map and find nodes
    unordered_map<TreeNode*, TreeNode*> parent;
    TreeNode* startNode = nullptr;
    TreeNode* destNode = nullptr;
    
    queue<TreeNode*> q;
    q.push(root);
    parent[root] = nullptr;
    
    while (!q.empty()) {
        TreeNode* node = q.front();
        q.pop();
        
        if (node->val == startValue) startNode = node;
        if (node->val == destValue) destNode = node;
        
        if (node->left) {
            parent[node->left] = node;
            q.push(node->left);
        }
        if (node->right) {
            parent[node->right] = node;
            q.push(node->right);
        }
    }
    
    // Build path from start to root
    vector<TreeNode*> startPath;
    TreeNode* curr = startNode;
    while (curr) {
        startPath.push_back(curr);
        curr = parent[curr];
    }
    
    // Build path from dest to root
    vector<TreeNode*> destPath;
    curr = destNode;
    while (curr) {
        destPath.push_back(curr);
        curr = parent[curr];
    }
    
    // Find LCA (last common node in both paths)
    int i = startPath.size() - 1, j = destPath.size() - 1;
    while (i >= 0 && j >= 0 && startPath[i] == destPath[j]) {
        i--; j--;
    }
    
    // Build result
    string result(startPath.size() - 1 - i, 'U');
    
    // Build path from LCA to dest
    for (int k = j; k >= 0; k--) {
        TreeNode* node = destPath[k];
        TreeNode* par = destPath[k + 1];
        if (par->left == node) result += 'L';
        else result += 'R';
    }
    
    return result;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

**Advantages:**
- No recursion stack
- Explicit parent tracking
- Easier to understand for some

**Disadvantages:**
- More code
- Requires building parent map

## Related Problems
- Lowest Common Ancestor of a Binary Tree
- Path Sum
- Binary Tree Paths
- Step-By-Step Directions From a Binary Tree Node to Another (this problem)



