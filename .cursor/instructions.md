# Project Instructions for Cursor AI

## Repository Purpose
This repository documents Google interview questions with comprehensive solutions in **C++**.

## Key Guidelines

### 1. Language Requirement
- **MUST use C++** for all code implementations
- Never use Python, Java, JavaScript, or any other language
- Use modern C++ standards (C++17 or later)

### 2. Code Structure
```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    vector<int> solutionName(vector<int>& params) {
        // Implementation
        return {};
    }
};
```

### 3. Documentation Template - REQUIRED FORMAT ORDER
Every question file **MUST follow this exact structure and order**:

1. **# Question Title** (H1)

2. **## Problem Statement** (H2)
   - Clear description of the problem
   - What is being asked
   - Any important assumptions

3. **## Examples** (H2)
   - At least 2-3 examples
   - Format each as `### Example 1`, `### Example 2`, etc.
   - Each example must have:
     - Input
     - Output
     - Explanation

4. **## Constraints** (H2)
   - List all constraints
   - Input size limits
   - Value ranges

5. **## Approach** (H2)
   - Must include 4 steps (all as H3):
     - `### Step 1: Understanding the Problem`
     - `### Step 2: Identifying the Strategy`
     - `### Step 3: Algorithm Design`
     - `### Step 4: Edge Cases`

6. **## Solution** (H2)
   - Contains two H3 subsections:
     - `### Code Implementation` - C++ code in ```cpp code block
     - `### Explanation` - Line-by-line or section explanation

7. **## Complexity Analysis** (H2)
   - `### Time Complexity` (H3)
   - `### Space Complexity` (H3)

8. **## Key Insights** (H2)
   - Important takeaways
   - Learning points

9. **## Alternative Approaches** (H2) [Optional]
   - Other possible solutions

10. **## Related Problems** (H2) [Optional]
    - Links to similar problems

**IMPORTANT**: Maintain this exact order. Do not rearrange sections.

### 4. File Naming
- Format: `##-problem-name.md`
- Example: `01-two-sum.md`, `02-valid-parentheses.md`

### 5. Code Quality
- Add clear comments
- Use meaningful variable names
- Include function documentation
- Explain complex logic
- Handle edge cases

### 6. When Creating New Questions
1. Use TEMPLATE.md as base
2. **Follow the exact format order**: Problem Statement → Examples → Constraints → Approach → Solution → Complexity Analysis → Key Insights
3. Write complete C++ solution in the Solution section
4. Include at least 2-3 examples with full details (input, output, explanation)
5. Break down Approach into exactly 4 steps
6. Add the question link to README.md
7. Follow naming convention: `##-problem-name.md`
8. Use proper markdown heading levels (H1, H2, H3)
9. Ensure all required sections are filled completely

### 7. When Modifying Code
- Keep C++ language
- Maintain consistent style
- Update explanations if needed
- Verify code correctness

## Example Code Pattern
```cpp
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    /**
     * Brief description
     * @param nums Input vector
     * @param target Target value
     * @return Result vector
     */
    vector<int> solve(vector<int>& nums, int target) {
        unordered_map<int, int> map;
        
        for (int i = 0; i < nums.size(); i++) {
            // Logic here
        }
        
        return {};
    }
};
```

## Remember
- Always use C++ for code (never Python, Java, etc.)
- Follow the exact format order: Problem Statement → Examples → Constraints → Approach → Solution → Complexity Analysis → Key Insights
- Use proper markdown heading levels (H1 for title, H2 for main sections, H3 for subsections)
- Include at least 2-3 examples with inputs, outputs, and explanations
- Break down Approach into exactly 4 steps
- Add comprehensive explanations
- Include complexity analysis (time and space)
- Keep code well-commented
- Reference FORMAT.md for detailed format guidelines


