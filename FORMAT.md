# Markdown File Format Guide

This document explains the standard format that should be followed for all question markdown files.

## Required Format Structure

Each question markdown file should follow this exact order:

### 1. **Question Title** (H1)
```
# Question Name
```

### 2. **Problem Statement** (H2)
- Clear description of the problem
- What is being asked
- Any important assumptions

### 3. **Examples** (H2)
- At least 2-3 examples
- Each example should have:
  - Input
  - Output
  - Explanation

### 4. **Constraints** (H2)
- List all constraints
- Input size limits
- Value ranges
- Any special conditions

### 5. **Approach** (H2)
Break down into 4 steps:
- **Step 1: Understanding the Problem**
- **Step 2: Identifying the Strategy**
- **Step 3: Algorithm Design**
- **Step 4: Edge Cases**

### 6. **Solution** (H2)
Contains two subsections:
- **Code Implementation** (H3) - C++ code in code block
- **Explanation** (H3) - Line-by-line or section explanation

### 7. **Complexity Analysis** (H2)
- **Time Complexity** (H3)
- **Space Complexity** (H3)

### 8. **Key Insights** (H2)
- Important takeaways
- Learning points
- Pattern recognition

### 9. **Alternative Approaches** (H2) [Optional]
- Other possible solutions
- Different time/space trade-offs

### 10. **Related Problems** (H2) [Optional]
- Links to similar problems
- Variations of the problem

## Quick Format Summary

```
# Question Title

## Problem Statement
[Description]

## Examples
[Multiple examples with inputs/outputs]

## Constraints
[List of constraints]

## Approach
### Step 1: Understanding the Problem
### Step 2: Identifying the Strategy
### Step 3: Algorithm Design
### Step 4: Edge Cases

## Solution
### Code Implementation
[C++ code]
### Explanation
[Code explanation]

## Complexity Analysis
### Time Complexity
### Space Complexity

## Key Insights
[Takeaways]

## Alternative Approaches
[Other solutions - optional]

## Related Problems
[Similar problems - optional]
```

## Important Notes

- **All code must be in C++**
- Use proper markdown formatting
- Code blocks should use `cpp` syntax highlighting
- Keep explanations clear and educational
- Follow the exact order shown above


