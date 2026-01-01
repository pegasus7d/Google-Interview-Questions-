# Longest Valid Word

## Problem Statement

Given a dictionary (set) of strings, find the **longest valid word** in the dictionary.

**Definition of Valid Word:**
A word is valid if you can:
1. Remove exactly **one character** at a time
2. Each intermediate string must exist in the dictionary
3. Eventually reach a string of **length 1**

**Goal:** Return the longest valid word from the dictionary.

**Input Format:**
- `words`: Vector of strings (dictionary)

**Output:** The longest valid word, or empty string if no valid word exists.

## Examples

### Example 1
```
Input: words = ["string", "sring", "sing", "wording", "ing", "ng", "g"]

Valid chain for "string":
string → sring → sing → ing → ng → g
(All intermediate words exist in dictionary)

Valid chain for "wording":
wording → (cannot form valid chain, no valid deletion)

Output: "string" (length = 6)
```

### Example 2
```
Input: words = ["a", "ab", "abc", "abcd"]

Valid chains:
- "a": length 1 (base case)
- "ab": ab → a (length 2)
- "abc": abc → ab → a (length 3)
- "abcd": abcd → abc → ab → a (length 4)

Output: "abcd" (length = 4)
```

### Example 3
```
Input: words = ["a", "b", "c", "ab", "bc"]

Valid chains:
- "a": length 1
- "b": length 1
- "c": length 1
- "ab": ab → a or ab → b (length 2)
- "bc": bc → b or bc → c (length 2)

Output: "ab" or "bc" (both have length 2)
```

## Constraints
- `1 <= words.length <= 1000`
- `1 <= words[i].length <= 100`
- All words consist of lowercase English letters
- Words are distinct

## Approach

### Step 1: Understanding the Problem
We need to find the longest word that can be reduced by removing one character at a time, with all intermediate words in the dictionary, until reaching length 1.

**Key Insight:** This forms a directed graph where:
- **Node** = word in dictionary
- **Edge** = removing one character to form another word (if it exists in dictionary)
- We want the **longest path** ending at a word of length 1

### Step 2: Identifying the Strategy
This is a **graph traversal + dynamic programming** problem:
- **DFS** to explore all deletion paths
- **Memoization** to avoid recomputing the same word multiple times
- Each word's result depends on smaller words formed by deletion

**Why not Trie?**
- Trie helps with **prefix matching**, not character deletion
- We're deleting characters at **arbitrary positions**, not working with prefixes
- Each deletion creates a new arbitrary string, not a sequential prefix
- Graph + DP is the correct mental model

### Step 3: Algorithm Design
1. **Store dictionary** in a hash set for O(1) lookup
2. **Define recursive function** `solve(word)`:
   - Returns length of longest valid deletion chain starting from `word`
   - Base case: If no valid deletion exists, return 1 (word itself)
   - Try removing each character at every position
   - If resulting word exists in dictionary, recursively solve it
   - Return maximum chain length
3. **Add memoization**: Store `solve(word)` result to avoid recomputation
4. **For each word** in dictionary, compute its chain length
5. **Return** the word with maximum chain length

### Step 4: Edge Cases
- Single character words: Valid by definition (length 1)
- Words with no valid deletion path: Return length 1
- Multiple words with same maximum length: Return any one
- Words that cannot reach length 1: Not considered valid

## Solution

### Recursive Function Explanation

**What does `solve(word)` do?**
- Returns the **length** of the longest valid deletion chain starting from `word`
- In simple terms: "If I start from this word and keep deleting one character at a time (with all intermediates in dictionary), what is the maximum number of words I can form?"

**Recurrence Relation:**
```
solve(word) = 1 + max(solve(nextWord))
```
where `nextWord` is formed by removing one character from `word` and exists in dictionary.

**Base Case:**
If no valid deletion exists, `solve(word) = 1` (the word itself counts).

### Step 1: Pure Recursion (No Memoization)

```cpp
#include <vector>
#include <unordered_set>
#include <string>
#include <algorithm>
using namespace std;

class Solution {
public:
    /**
     * Pure recursive solution (no memoization)
     * Returns length of longest valid deletion chain starting from word
     */
    int solve(const string& word, unordered_set<string>& dict) {
        int best = 1;  // At least the word itself
        
        // Try removing one character at each position
        for (int i = 0; i < word.size(); i++) {
            // Remove character at position i
            string next = word.substr(0, i) + word.substr(i + 1);
            
            // If the new word exists in dictionary, recursively solve it
            if (dict.count(next)) {
                best = max(best, 1 + solve(next, dict));
            }
        }
        
        return best;
    }
    
    string longestValidWord(vector<string>& words) {
        // Build dictionary set
        unordered_set<string> dict(words.begin(), words.end());
        
        string answer = "";
        int maxLen = 0;
        
        // Try every word as starting point
        for (const string& w : words) {
            int len = solve(w, dict);
            if (len > maxLen) {
                maxLen = len;
                answer = w;
            }
        }
        
        return answer;
    }
};
```

**Problem with this approach:**
- Same word is computed multiple times
- Example: `"ing"` might be reached from both `"sing"` and `"wording"`
- Time complexity becomes **exponential**

### Step 2: Add Memoization (Optimized Solution)

```cpp
#include <vector>
#include <unordered_set>
#include <unordered_map>
#include <string>
#include <algorithm>
using namespace std;

class Solution {
private:
    unordered_set<string> dict;
    unordered_map<string, int> memo;
    
    /**
     * Recursive function with memoization
     * Returns length of longest valid deletion chain starting from word
     * 
     * @param word Current word being processed
     * @return Length of longest valid chain starting from word
     */
    int solve(const string& word) {
        // If already computed, return from memo
        if (memo.count(word)) {
            return memo[word];
        }
        
        int best = 1;  // At least the word itself
        
        // Try removing one character at each position
        for (int i = 0; i < word.size(); i++) {
            // Remove character at position i
            string next = word.substr(0, i) + word.substr(i + 1);
            
            // If the new word exists in dictionary, recursively solve it
            if (dict.count(next)) {
                best = max(best, 1 + solve(next));
            }
        }
        
        // Store result in memo before returning
        memo[word] = best;
        return best;
    }
    
public:
    /**
     * Find the longest valid word in the dictionary
     * 
     * @param words Vector of strings (dictionary)
     * @return Longest valid word
     */
    string longestValidWord(vector<string>& words) {
        // Reset for multiple calls
        dict.clear();
        memo.clear();
        
        // Build dictionary set for O(1) lookup
        for (const string& w : words) {
            dict.insert(w);
        }
        
        string answer = "";
        int maxLen = 0;
        
        // Try every word as starting point
        for (const string& w : words) {
            int len = solve(w);
            if (len > maxLen) {
                maxLen = len;
                answer = w;
            }
        }
        
        return answer;
    }
};
```

### Explanation

**Recursive Function (`solve`):**
- **Line 20-23**: **Memoization check** - If we've already computed this word, return stored result
- **Line 25**: Initialize `best = 1` (word itself counts as length 1)
- **Line 27-33**: Try removing each character:
  - **Line 29**: Create new word by removing character at position `i`
  - **Line 31**: If new word exists in dictionary, recursively solve it
  - **Line 32**: Update `best` with maximum chain length
- **Line 35-36**: Store result in memo and return

**Main Function (`longestValidWord`):**
- **Line 45-48**: Build dictionary set for O(1) lookup
- **Line 50-58**: For each word, compute its chain length and track maximum
- **Line 57**: Return word with maximum chain length

**Recursion Tree Example:**
```
solve("string")
│
├── solve("sring")   [remove 't']
│   │
│   ├── solve("sing")   [remove 'r']
│   │   │
│   │   ├── solve("ing")   [remove 's']
│   │   │   │
│   │   │   ├── solve("ng")   [remove 'i']
│   │   │   │   │
│   │   │   │   ├── solve("g")   [remove 'n']
│   │   │   │   │   └── return 1  (base case)
│   │   │   │   │
│   │   │   │   └── return 2  ("ng" → "g")
│   │   │   │
│   │   │   └── return 3  ("ing" → "ng" → "g")
│   │   │
│   │   └── return 4  ("sing" → "ing" → "ng" → "g")
│   │
│   └── return 5  ("sring" → "sing" → "ing" → "ng" → "g")
│
└── return 6  ("string" → "sring" → "sing" → "ing" → "ng" → "g")
```

**Memo Table Filling Order:**
```
memo["g"] = 1
memo["ng"] = 2
memo["ing"] = 3
memo["sing"] = 4
memo["sring"] = 5
memo["string"] = 6
```

## Complexity Analysis

### Time Complexity
- **O(N × L²)**: 
  - `N` = number of words
  - `L` = maximum word length
  - For each word, we try `L` deletions
  - Each string creation is `O(L)`
  - Each word is computed **once** due to memoization
  - **Overall**: O(N × L²)

### Space Complexity
- **O(N × L)**: 
  - Dictionary set: O(N × L)
  - Memo map: O(N × L) in worst case
  - Recursion stack: O(L) (depth of longest chain)
  - **Overall**: O(N × L)

### Comparison

| Approach | Time Complexity | Space Complexity |
|----------|----------------|------------------|
| Pure Recursion | O(2^L) exponential | O(L) |
| With Memoization | O(N × L²) | O(N × L) |

## Key Insights
- **Graph Model**: Think of words as nodes, deletions as edges - we want the longest path
- **Memoization is Essential**: Without it, same words are recomputed exponentially
- **Why Not Trie**: Trie helps with prefixes, but we're deleting characters at arbitrary positions
- **Recursive Structure**: Each word's answer depends on smaller words - perfect for DP
- **Base Case**: Words with no valid deletion have chain length 1
- **Optimal Substructure**: Longest chain from a word = 1 + max(longest chain from all valid smaller words)

## Alternative Approaches

### Iterative DP (Bottom-Up)

```cpp
string longestValidWordIterative(vector<string>& words) {
    unordered_set<string> dict(words.begin(), words.end());
    unordered_map<string, int> dp;
    
    // Sort words by length (shortest first)
    sort(words.begin(), words.end(), 
         [](const string& a, const string& b) {
             return a.size() < b.size();
         });
    
    int maxLen = 0;
    string answer = "";
    
    // Process words from shortest to longest
    for (const string& word : words) {
        int best = 1;  // At least the word itself
        
        // Try removing one character
        for (int i = 0; i < word.size(); i++) {
            string next = word.substr(0, i) + word.substr(i + 1);
            if (dict.count(next) && dp.count(next)) {
                best = max(best, 1 + dp[next]);
            }
        }
        
        dp[word] = best;
        
        if (best > maxLen) {
            maxLen = best;
            answer = word;
        }
    }
    
    return answer;
}
```

**Time Complexity:** O(N × L²)  
**Space Complexity:** O(N × L)

**Advantages:**
- No recursion stack overhead
- Processes words in order of length
- More cache-friendly

### Return Actual Chain (Not Just Length)

```cpp
vector<string> getLongestChain(vector<string>& words) {
    unordered_set<string> dict(words.begin(), words.end());
    unordered_map<string, vector<string>> memo;
    
    function<vector<string>(const string&)> solve = [&](const string& word) {
        if (memo.count(word)) return memo[word];
        
        vector<string> best = {word};
        
        for (int i = 0; i < word.size(); i++) {
            string next = word.substr(0, i) + word.substr(i + 1);
            if (dict.count(next)) {
                vector<string> chain = solve(next);
                if (chain.size() + 1 > best.size()) {
                    best = {word};
                    best.insert(best.end(), chain.begin(), chain.end());
                }
            }
        }
        
        memo[word] = best;
        return best;
    };
    
    vector<string> longestChain;
    for (const string& w : words) {
        vector<string> chain = solve(w);
        if (chain.size() > longestChain.size()) {
            longestChain = chain;
        }
    }
    
    return longestChain;
}
```

## Related Problems
- Word Ladder
- Longest String Chain
- Delete Operation for Two Strings
- Edit Distance



