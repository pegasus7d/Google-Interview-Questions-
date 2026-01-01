# Google Interview Questions

This repository contains solutions to Google interview questions with detailed approaches and explanations. All solutions are implemented in **C++**.

## Structure

Each question is documented in a separate markdown file following this **exact format order**:
1. **Problem Statement**: Clear description of the problem
2. **Examples**: Input/output examples (at least 2-3)
3. **Constraints**: List all constraints
4. **Approach**: Step-by-step breakdown (4 steps: Understanding, Strategy, Algorithm Design, Edge Cases)
5. **Solution**: C++ code implementation with explanation
6. **Complexity Analysis**: Time and Space complexity
7. **Key Insights**: Important takeaways from the problem
8. **Alternative Approaches**: Other solutions (optional)
9. **Related Problems**: Similar problems (optional)

**See [FORMAT.md](FORMAT.md) for detailed format guidelines.**

## How to Use

1. Create a new markdown file for each question (e.g., `01-two-sum.md`)
2. Follow the template structure provided in `TEMPLATE.md`
3. **Maintain the exact format order**: Problem Statement → Examples → Constraints → Approach → Solution → Complexity Analysis → Key Insights
4. Add your C++ solution code with proper formatting
5. Include at least 2-3 examples with inputs, outputs, and explanations
6. Update this README with links to all questions

## Questions

- [Format Guide](FORMAT.md) - Detailed format structure and guidelines
- [Template](TEMPLATE.md) - Template for documenting questions
- [Q1: Maximum Distance with Energy Constraint](01-boat-energy.md) - Telephonic Round: Maximize distance traveled with energy constraints on a boat
- [Q2: Horizontal Line Dividing Squares in Half](02-square-division.md) - Find horizontal line that divides total square area into two equal halves using sweep line algorithm
- [Q3: Shortest Path to Favorite City](03-shortest-path-to-favorite-city.md) - Round 1: Find favorite city reachable from source in minimum time using Dijkstra's algorithm
- [Q4: Highest Score Anagram](04-highest-score-anagram.md) - Round 2: Find anagram with highest score using precomputation and hash grouping
- [Q5: Largest Rectangle from Points](05-largest-rectangle-from-points.md) - Round 3: Find 4 points forming rectangle with largest area (axis-aligned and rotated)
- [Q6: Maximum Path Sum Leaf Nodes](06-maximum-path-sum-leaf-nodes.md) - First Round: Find all leaf nodes with maximum root-to-leaf path sum using DFS
- [Q7: Longest Valid Word](07-longest-valid-word.md) - First Round: Find longest word that can be reduced by removing one character at a time using DFS with memoization
- [Q8: Maximum Island Triplet Sum](08-maximum-island-triplet.md) - Round 1 DSA: Find 3 islands forming path A-B-C with maximum sum using graph traversal
- [Q9: Koko Eating Bananas](09-koko-eating-bananas.md) - LeetCode 875: Find minimum eating speed to finish all bananas within h hours using binary search
- [Q10: Step-By-Step Directions](10-step-by-step-directions.md) - LeetCode 2096: Find shortest path directions between two nodes using LCA and path finding
- [Q11: Largest Square Submatrix](11-largest-square-submatrix.md) - Find position of square submatrix of all 1's with size >= k using dynamic programming
- [Q12: First Expired RPC](12-first-expired-rpc.md) - Round 1: Find first time when any RPC expires based on timeout using hash maps
- [Q13: Rearrange Neighbourhoods](13-rearrange-neighbourhoods.md) - Round 2: Rearrange houses in neighbourhoods to be sorted with no duplicates using greedy approach with multiset
- [Q14: Square of 1's with Constraint](14-square-of-ones-with-constraint.md) - Find square of 1's with side length >= ⌈√n⌉ using recursive DP with memoization
- [Q15: Maximum Min Plus Max Subarray](15-maximum-min-plus-max-subarray.md) - Find subarray with maximum (min + max) value by checking adjacent pairs

<!-- Add more questions here as you solve them -->

---

**Note**: This repository is for educational purposes and interview preparation.

