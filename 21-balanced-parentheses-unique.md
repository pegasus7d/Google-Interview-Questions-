# Balanced Parentheses with Unique Pairing

## Problem Statement

You are given a string `s` containing only the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`. Your task is to determine if the input string is valid according to the following rules:

1. **Open brackets must be closed by the same type of brackets.**
2. **Open brackets must be closed in the correct order.**
3. **Unique Pairing Constraint**: Each open bracket must be paired with a corresponding closing bracket in a unique way. That is, for each open bracket, there should not be multiple options for the corresponding closing bracket.

**What "Unique Pairing" Means:**
- When closing a bracket, there must be exactly **one** opening bracket of that type on the stack
- If there are multiple opening brackets of the same type, it's ambiguous which one to close
- This ensures each pairing is unambiguous

**Input Format:**
- `s`: String containing only parentheses characters

**Output:** Return `true` if the string is valid, `false` otherwise.

## Examples

### Example 1
```
Input: s = "()"
Output: true

Explanation:
- '(' is pushed onto stack
- ')' closes '(' (only one '(' on stack, so unique)
- Stack is empty → valid
```

### Example 2
```
Input: s = "()[]{}"
Output: true

Explanation:
- '(' → push, count['('] = 1
- ')' → closes '(' (count['('] = 1, unique) → pop
- '[' → push, count['['] = 1
- ']' → closes '[' (count['['] = 1, unique) → pop
- '{' → push, count['{'] = 1
- '}' → closes '{' (count['{'] = 1, unique) → pop
- Stack empty → valid
```

### Example 3
```
Input: s = "([)]"
Output: false

Explanation:
- '(' → push, count['('] = 1
- '[' → push, count['['] = 1
- ')' → tries to close '('
  - Top of stack is '[' (not '(') → invalid order
- Returns false
```

### Example 4
```
Input: s = "((()))"
Output: false

Explanation:
- '(' → push, count['('] = 1
- '(' → push, count['('] = 2
- ')' → tries to close '('
  - count['('] = 2 > 1 → ambiguous (which '(' to close?)
  - Returns false
```

### Example 5
```
Input: s = "({})"
Output: true

Explanation:
- '(' → push, count['('] = 1
- '{' → push, count['{'] = 1
- '}' → closes '{' (count['{'] = 1, unique) → pop
- ')' → closes '(' (count['('] = 1, unique) → pop
- Stack empty → valid
```

## Constraints
- `1 <= s.length <= 10^4`
- `s` consists of only `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`

## Approach

### Step 1: Understanding the Problem
This extends the classic balanced parentheses problem with a **uniqueness constraint**:

**Standard Rules:**
- Matching brackets must be of the same type
- Brackets must be closed in correct order (LIFO)

**Unique Constraint:**
- When closing a bracket, there must be exactly **one** opening bracket of that type on the stack
- If multiple opening brackets of the same type exist, it's ambiguous which one to close
- This prevents ambiguous pairings

**Key Insight:** Track the count of each opening bracket type. When closing, if count > 1, it's ambiguous.

### Step 2: Identifying the Strategy
**Stack-Based Approach with Count Tracking:**
1. Use stack to track opening brackets (maintains order)
2. Use map to count occurrences of each opening bracket type
3. When opening: push to stack, increment count
4. When closing:
   - Check stack is not empty
   - Check top matches expected opening
   - **Check count == 1** (unique constraint)
   - If all pass: pop from stack, decrement count

**Why This Works:**
- Stack ensures correct order (LIFO)
- Count ensures uniqueness (only one of each type at closing time)
- Both conditions together guarantee valid unique pairing

### Step 3: Algorithm Design
1. **Initialize**: Stack, close-to-open mapping, open set, count map
2. **For each character**:
   - **If opening bracket**:
     - Push to stack
     - Increment count for this bracket type
   - **If closing bracket**:
     - Check stack not empty
     - Check top matches expected opening
     - **Check count == 1** (unique constraint)
     - If all pass: pop, decrement count
     - Otherwise: return false
3. **Return**: `stack.empty()`

### Step 4: Edge Cases
- Empty string: Return true (no brackets to validate)
- Only opening brackets: Return false (stack not empty)
- Only closing brackets: Return false (stack empty when closing)
- Mismatched types: Return false (top doesn't match)
- Multiple same opening: Return false (count > 1 when closing)
- Nested valid: Works correctly (count resets as brackets close)

## Solution

### Code Implementation

```cpp
#include <string>
#include <stack>
#include <unordered_map>
#include <unordered_set>
using namespace std;

class Solution {
public:
    /**
     * Check if parentheses string is valid with unique pairing constraint
     * 
     * @param s String containing only parentheses characters
     * @return true if valid, false otherwise
     */
    bool isValidUnique(const string& s) {
        stack<char> st;
        
        // Map closing brackets to their corresponding opening brackets
        unordered_map<char, char> closeToOpen;
        closeToOpen[')'] = '(';
        closeToOpen[']'] = '[';
        closeToOpen['}'] = '{';
        
        // Set of opening brackets for fast lookup
        unordered_set<char> open;
        open.insert('(');
        open.insert('[');
        open.insert('{');
        
        // Count of each opening bracket type currently on stack
        unordered_map<char, int> openCount;
        
        for (auto ch : s) {
            if (open.find(ch) != open.end()) {
                // Opening bracket: push to stack and increment count
                st.push(ch);
                openCount[ch]++;
            } else {
                // Closing bracket: validate and process
                
                // Check 1: Stack must not be empty
                if (st.empty()) {
                    return false;
                }
                
                // Check 2: Top of stack must match expected opening
                char expectedOpening = closeToOpen[ch];
                if (st.top() != expectedOpening) {
                    return false;
                }
                
                // Check 3: Unique constraint - only one opening of this type
                if (openCount[expectedOpening] > 1) {
                    return false;  // Ambiguous: multiple options to close
                }
                
                // All checks passed: pop and decrement count
                st.pop();
                openCount[expectedOpening]--;
            }
        }
        
        // Stack must be empty (all brackets matched)
        return st.empty();
    }
};
```

### Explanation
- **Line 18-22**: Initialize data structures:
  - **Stack**: Tracks opening brackets in order
  - **closeToOpen**: Maps `')'` → `'('`, `']'` → `'['`, `'}'` → `'{'`
  - **open**: Set of opening brackets for fast lookup
  - **openCount**: Counts occurrences of each opening bracket type
  
- **Line 24-47**: Process each character:
  - **Line 25-28**: **Opening bracket**:
    - Push to stack
    - Increment count for this bracket type
    
  - **Line 29-46**: **Closing bracket**:
    - **Line 32-34**: Check stack is not empty
    - **Line 36-39**: Check top matches expected opening bracket
    - **Line 41-44**: **Unique constraint check**:
      - If `openCount[expectedOpening] > 1`, there are multiple opening brackets of this type
      - This makes it ambiguous which one to close
      - Return false
    - **Line 46-47**: All checks passed:
      - Pop from stack
      - Decrement count
      
- **Line 50**: Return true only if stack is empty (all brackets matched)

**Why Unique Constraint Works:**
- When `openCount[expectedOpening] == 1`, there's exactly one opening bracket of that type
- This means there's no ambiguity about which opening bracket to close
- When `openCount[expectedOpening] > 1`, multiple openings exist, making pairing ambiguous

**Example Walkthrough:**
```
s = "({})"

Step 1: '(' → push, count['('] = 1
Stack: ['(']

Step 2: '{' → push, count['{'] = 1
Stack: ['(', '{']

Step 3: '}' → closing
  - Stack not empty ✓
  - Top is '{', expected '{' ✓
  - count['{'] = 1 (unique) ✓
  - Pop '{', count['{'] = 0
Stack: ['(']

Step 4: ')' → closing
  - Stack not empty ✓
  - Top is '(', expected '(' ✓
  - count['('] = 1 (unique) ✓
  - Pop '(', count['('] = 0
Stack: []

Result: true (stack empty)
```

## Complexity Analysis

### Time Complexity
- **O(n)**: 
  - Process each character once: O(n)
  - Stack operations (push/pop): O(1) each
  - Map/set operations: O(1) each
  - **Overall**: O(n)

### Space Complexity
- **O(n)**: 
  - Stack: O(n) in worst case (all opening brackets)
  - Maps and sets: O(1) (fixed number of bracket types)
  - **Overall**: O(n)

## Key Insights
- **Stack for Order**: Maintains LIFO order for bracket matching
- **Count for Uniqueness**: Tracks how many of each opening bracket type are on stack
- **Unique Constraint**: `count == 1` ensures unambiguous pairing
- **Three Checks**: Stack empty, type match, uniqueness
- **Ambiguity Prevention**: Multiple same-type openings make pairing ambiguous
- **Standard + Unique**: Extends classic problem with additional constraint

## Alternative Approaches

### Without Count Map (Incorrect for Unique Constraint)

```cpp
bool isValidStandard(const string& s) {
    stack<char> st;
    unordered_map<char, char> closeToOpen = {
        {')', '('}, {']', '['}, {'}', '{'}
    };
    
    for (char ch : s) {
        if (closeToOpen.find(ch) == closeToOpen.end()) {
            st.push(ch);  // Opening bracket
        } else {
            if (st.empty() || st.top() != closeToOpen[ch]) {
                return false;
            }
            st.pop();
        }
    }
    
    return st.empty();
}
```

**Why This Doesn't Work:**
- Doesn't check uniqueness constraint
- Would return true for `"((()))"` (should be false)
- Missing the count-based uniqueness check

### Recursive Approach (More Complex)

```cpp
bool isValidUniqueRecursive(const string& s, int& idx, char expected) {
    // More complex recursive solution
    // Less efficient and harder to understand
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n) for recursion stack

**Why Stack is Better:**
- More intuitive
- Easier to implement
- Same complexity

## Related Problems
- Valid Parentheses (LeetCode 20) - Standard version
- Generate Parentheses
- Remove Invalid Parentheses
- Longest Valid Parentheses

