# Maximum Path Sum Leaf Nodes

## Problem Statement

Given a binary tree, for every root-to-leaf path, compute the sum of node values along the path. Return **all leaf nodes** whose root-to-leaf path sum is **maximum**.

**Input:** Root of a binary tree

**Output:** Vector of all leaf node values that have the maximum path sum from root to leaf.

## Examples

### Example 1
```
Input:
        5
       / \
      3   8
     /     \
    2       10
   /
  1

Root-to-leaf paths:
- 5 → 3 → 2 → 1: sum = 11
- 5 → 8 → 10: sum = 23

Maximum sum = 23
Output: [10]
```

### Example 2
```
Input:
        1
       / \
      2   3
     /   / \
    4   5   6

Root-to-leaf paths:
- 1 → 2 → 4: sum = 7
- 1 → 3 → 5: sum = 9
- 1 → 3 → 6: sum = 10

Maximum sum = 10
Output: [6]
```

### Example 3
```
Input:
        10
       /  \
      5    5
     / \  / \
    2   3 2  3

Root-to-leaf paths:
- 10 → 5 → 2: sum = 17
- 10 → 5 → 3: sum = 18
- 10 → 5 → 2: sum = 17
- 10 → 5 → 3: sum = 18

Maximum sum = 18
Output: [3, 3]  (both leaves with sum 18)
```

## Constraints
- `1 <= number of nodes <= 10^4`
- `-10^3 <= node.val <= 10^3`
- Tree may not be balanced
- Multiple leaves can have the same maximum sum

## Approach

### Step 1: Understanding the Problem
We need to:
1. Traverse all root-to-leaf paths
2. Calculate the sum for each path
3. Find the maximum sum among all paths
4. Return all leaf nodes that have this maximum sum

**Key Insight:** A node is a leaf node if and only if both its left and right children are `nullptr` (or `NULL`).

### Step 2: Identifying the Strategy
This is a tree traversal problem where we need to:
- Visit every node (DFS/BFS)
- Track the current path sum
- Identify leaf nodes
- Maintain maximum sum and corresponding leaf nodes

**DFS (Preorder Traversal)** is suitable because:
- We can maintain current path sum as we traverse
- We naturally reach leaf nodes during traversal
- Easy to update maximum and collect results

### Step 3: Algorithm Design
1. **Initialize:**
   - `maxSum = INT_MIN` (global maximum path sum)
   - `resultLeaves = []` (list of leaf nodes with maximum sum)

2. **DFS Traversal:**
   - Start from root with `currentSum = 0`
   - At each node, add node value to `currentSum`
   - If node is leaf (`left == nullptr && right == nullptr`):
     - If `currentSum > maxSum`: Update `maxSum`, clear result, add current leaf
     - If `currentSum == maxSum`: Add current leaf to result
   - Recursively traverse left and right subtrees

3. **Return:** All leaf nodes with maximum path sum

### Step 4: Edge Cases
- Single node tree: Root itself is the leaf
- All paths have same sum: Return all leaves
- Negative values: Use `INT_MIN` for initial maximum
- Skewed tree: Still works, just longer paths

## Solution

### Code Implementation

```cpp
#include <vector>
#include <climits>
using namespace std;

// Definition for a binary tree node
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
private:
    int maxSum = INT_MIN;
    vector<int> resultLeaves;
    
    /**
     * DFS traversal to find leaf nodes with maximum path sum
     * 
     * @param node Current node being processed
     * @param currentSum Sum of values from root to current node
     */
    void dfs(TreeNode* node, int currentSum) {
        // Base case: null node
        if (!node) return;
        
        // Add current node's value to path sum
        currentSum += node->val;
        
        // Check if current node is a leaf node
        // A node is a leaf if both left and right children are null
        if (node->left == nullptr && node->right == nullptr) {
            // Leaf node reached
            if (currentSum > maxSum) {
                // Found a new maximum, update and reset result
                maxSum = currentSum;
                resultLeaves.clear();
                resultLeaves.push_back(node->val);
            } else if (currentSum == maxSum) {
                // Same maximum sum, add this leaf to result
                resultLeaves.push_back(node->val);
            }
            return;
        }
        
        // Recursively traverse left and right subtrees
        dfs(node->left, currentSum);
        dfs(node->right, currentSum);
    }
    
public:
    /**
     * Find all leaf nodes with maximum root-to-leaf path sum
     * 
     * @param root Root of the binary tree
     * @return Vector of leaf node values with maximum path sum
     */
    vector<int> maxSumLeafNodes(TreeNode* root) {
        // Reset for multiple calls
        maxSum = INT_MIN;
        resultLeaves.clear();
        
        // Start DFS from root with initial sum 0
        dfs(root, 0);
        
        return resultLeaves;
    }
};
```

### Explanation
- **Line 14-15**: Global variables to track maximum sum and result leaves
- **Line 17-44**: DFS function:
  - **Line 21**: Base case - return if node is null
  - **Line 24**: Add current node's value to path sum
  - **Line 27-38**: **Leaf node check**: `node->left == nullptr && node->right == nullptr`
    - If both children are null, it's a leaf node
    - **Line 29-33**: If current sum is greater than maximum, update maximum and reset result
    - **Line 34-36**: If current sum equals maximum, add this leaf to result
  - **Line 41-42**: Recursively traverse left and right subtrees
- **Line 48-58**: Main function:
  - **Line 51-52**: Reset global variables (important for multiple calls)
  - **Line 55**: Start DFS from root with initial sum 0
  - **Line 57**: Return all leaves with maximum path sum

**Why `node->left == nullptr && node->right == nullptr`?**
- A leaf node has **no children**
- If either `left` or `right` exists, the node is an internal node
- Both must be null for a node to be a leaf
- This works for all tree types: full, skewed, single-node

## Complexity Analysis

### Time Complexity
- **O(n)**: We visit each node exactly once during DFS traversal
- Where `n` is the number of nodes in the tree

### Space Complexity
- **O(h)**: Recursion stack space, where `h` is the height of the tree
- **O(k)**: Space for result vector, where `k` is the number of leaves with maximum sum
- **Overall**: O(h + k)
- **Worst case** (skewed tree): O(n) for recursion stack
- **Best case** (balanced tree): O(log n) for recursion stack

## Key Insights
- **Leaf Node Identification**: A node is a leaf if and only if both children are `nullptr`
- **DFS Traversal**: Preorder traversal naturally visits all root-to-leaf paths
- **Path Sum Tracking**: Maintain current sum as we traverse down the tree
- **Multiple Results**: When multiple leaves have the same maximum sum, collect all of them
- **Global vs Local**: Using global variables simplifies the solution (can also be done with references)

## Alternative Approaches

### Iterative DFS (Using Stack)

```cpp
vector<int> maxSumLeafNodesIterative(TreeNode* root) {
    if (!root) return {};
    
    int maxSum = INT_MIN;
    vector<int> resultLeaves;
    
    // Stack: {node, currentSum}
    stack<pair<TreeNode*, int>> st;
    st.push({root, 0});
    
    while (!st.empty()) {
        auto [node, currentSum] = st.top();
        st.pop();
        
        currentSum += node->val;
        
        // Check if leaf node
        if (!node->left && !node->right) {
            if (currentSum > maxSum) {
                maxSum = currentSum;
                resultLeaves.clear();
                resultLeaves.push_back(node->val);
            } else if (currentSum == maxSum) {
                resultLeaves.push_back(node->val);
            }
        } else {
            // Push children
            if (node->right) st.push({node->right, currentSum});
            if (node->left) st.push({node->left, currentSum});
        }
    }
    
    return resultLeaves;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(h) for stack

### Using References Instead of Global Variables

```cpp
void dfs(TreeNode* node, int currentSum, int& maxSum, vector<int>& result) {
    if (!node) return;
    
    currentSum += node->val;
    
    if (!node->left && !node->right) {
        if (currentSum > maxSum) {
            maxSum = currentSum;
            result.clear();
            result.push_back(node->val);
        } else if (currentSum == maxSum) {
            result.push_back(node->val);
        }
        return;
    }
    
    dfs(node->left, currentSum, maxSum, result);
    dfs(node->right, currentSum, maxSum, result);
}

vector<int> maxSumLeafNodes(TreeNode* root) {
    int maxSum = INT_MIN;
    vector<int> result;
    dfs(root, 0, maxSum, result);
    return result;
}
```

**Advantages:**
- No global variables
- Thread-safe
- Better for multiple calls

## Related Problems
- Binary Tree Maximum Path Sum
- Path Sum
- Path Sum II
- Sum Root to Leaf Numbers
- Binary Tree Paths


