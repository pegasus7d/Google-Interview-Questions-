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

### 3. Documentation Template
Every question file must include:
- Problem Statement
- Examples (at least 2-3)
- Constraints
- Approach (4 steps)
- Solution (C++ code)
- Complexity Analysis
- Key Insights
- Alternative Approaches (if applicable)
- Related Problems

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
2. Write complete C++ solution
3. Add to README.md index
4. Follow naming convention
5. Ensure all sections are filled

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
- Always use C++ for code
- Follow the template structure
- Add comprehensive explanations
- Include complexity analysis
- Keep code well-commented

