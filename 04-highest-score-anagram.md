# Highest Score Anagram

## Problem Statement

Given:
- A dictionary of words
- A function `getHash(string word)` that returns the hash of a word (O(1))
- A function `getScore(string word)` that returns the sum of absolute differences between adjacent characters (O(len(word)))
- **Important**: All anagrams of a word have the same hash

Implement a function `findAnagram(string word)` that:
- Returns an **anagram** of the given word (from the dictionary) with the **highest score**
- The returned word must be **different** from the input word
- If multiple anagrams have the same highest score, return any one

**Example:**
- Words: `["cat", "tac", "dog", "god", "cow"]`
- `getHash("cat") == getHash("tac")` (both are anagrams)
- `getScore("cat") = |'a' - 'c'| + |'t' - 'a'| = 2 + 19 = 21`
- `getScore("tac") = |'a' - 't'| + |'c' - 'a'| = 19 + 2 = 21`
- For word `"cat"`, `findAnagram("cat")` should return `"tac"` (if it has higher or equal score)

## Examples

### Example 1
```
Input: words = ["cat", "tac", "dog", "god", "cow"]
findAnagram("cat")

Hash of "cat" = getHash("cat")
Anagrams with same hash: "cat", "tac"
- getScore("cat") = |'a'-'c'| + |'t'-'a'| = 2 + 19 = 21
- getScore("tac") = |'a'-'t'| + |'c'-'a'| = 19 + 2 = 21

Since "cat" == "cat" (input), we skip it and return "tac"
Output: "tac"
```

### Example 2
```
Input: words = ["cat", "tac", "act", "dog", "god"]
findAnagram("cat")

Anagrams: "cat", "tac", "act"
- getScore("cat") = 21
- getScore("tac") = 21
- getScore("act") = |'c'-'a'| + |'t'-'c'| = 2 + 17 = 19

Skip "cat", best among others is "tac" (score 21)
Output: "tac"
```

### Example 3
```
Input: words = ["dog", "god"]
findAnagram("dog")

Anagrams: "dog", "god"
- getScore("dog") = |'o'-'d'| + |'g'-'o'| = 11 + 8 = 19
- getScore("god") = |'o'-'g'| + |'d'-'o'| = 8 + 11 = 19

Skip "dog", return "god"
Output: "god"
```

## Constraints
- `1 <= words.size() <= 10^5`
- `1 <= len(word) <= 100`
- All words consist of lowercase English letters
- `getHash()` is O(1) and returns same hash for anagrams
- `getScore()` is O(len(word))
- The input word to `findAnagram()` is guaranteed to be in the dictionary

## Approach

### Step 1: Understanding the Problem
We need to:
1. Find all anagrams of the given word (words with same hash)
2. Calculate score for each anagram
3. Return the anagram with highest score (excluding the word itself)

**Key Insight:** Since all anagrams have the same hash, we can group words by hash and precompute scores.

### Step 2: Identifying the Strategy
**Brute Force Approach:**
- For each query, iterate through all words
- Check if hash matches
- Calculate score
- Find maximum (excluding input word)
- **Time Complexity:** O(N × len(word)) per query

**Optimized Approach:**
- **Precomputation:** Group words by hash, calculate scores once
- Store best-scoring anagram for each hash
- **Query:** O(1) - just lookup and return (skip if it's the input word)

### Step 3: Algorithm Design
1. **Precomputation Phase:**
   - Create map: `hash → vector of (word, score) pairs`
   - For each word in dictionary:
     - Calculate hash
     - Calculate score
     - Store in map
   - For each hash group, keep track of the word with maximum score
2. **Query Phase:**
   - Get hash of input word
   - Lookup best-scoring word for that hash
   - If best word equals input word, we need the second best (but wait - we skip input word, so we only need the best that's not the input)

### Step 4: Edge Cases
- Input word is the only anagram: Return empty string
- All anagrams have same score: Return any one (except input)
- Input word has highest score: Return second highest (but we skip input, so return the best among others)

## Solution

### Brute Force Solution (Lame Solution)

```cpp
#include <vector>
#include <string>
#include <climits>
using namespace std;

class Solution {
public:
    vector<string> words = {"cat", "tac", "dog", "god", "cow"};

    // Given (hidden implementation)
    string getHash(string word);

    // Score function: sum of absolute differences between adjacent characters
    int getScore(const string& word) {
        int score = 0;
        for (int i = 1; i < word.size(); i++) {
            score += abs(word[i] - word[i - 1]);
        }
        return score;
    }

    // Brute force solution - O(N × len(word)) per query
    string findAnagram(string word) {
        string targetHash = getHash(word);
        int bestScore = -1;
        string bestWord = "";

        // Iterate through all words
        for (const string& w : words) {
            // Skip the word itself (must return different anagram)
            if (w == word) continue;

            // Check if it's an anagram (same hash)
            if (getHash(w) == targetHash) {
                int score = getScore(w);
                if (score > bestScore) {
                    bestScore = score;
                    bestWord = w;
                }
            }
        }

        return bestWord; // Empty string if no anagram found
    }
};
```

**Time Complexity:** O(N × len(word)) per query  
**Space Complexity:** O(1)

**Problems with this approach:**
- Recomputes `getHash()` for same words multiple times
- Recomputes `getScore()` for same words multiple times
- No caching - every query does full scan

### Optimized Solution with Precomputation

```cpp
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using namespace std;

// Comparator function for sorting by score (descending)
bool cmp(const pair<string, int>& a, const pair<string, int>& b) {
    return a.second > b.second;  // Higher score first
}

class Solution {
private:
    // Precomputed data: hash -> best scoring word (that's not the word itself)
    unordered_map<string, string> bestAnagram;
    
    // Precomputation method
    void precompute() {
        // Map: hash -> vector of (word, score) pairs
        unordered_map<string, vector<pair<string, int>>> info;
        
        // 1. Group words by hash and calculate scores
        for (const string& w : words) {
            string h = getHash(w);
            int s = getScore(w);
            
            // Store word and its score for this hash
            info[h].push_back({w, s});
        }
        
        // 2. For each hash group, find the best scoring word
        for (auto& [hash, vec] : info) {
            // Sort by score (descending) - highest score first
            sort(vec.begin(), vec.end(), cmp);
            
            // Store the best scoring word for this hash
            // Note: We store the best one, and during query we'll skip if it's the input word
            if (!vec.empty()) {
                bestAnagram[hash] = vec[0].first;
            }
        }
    }
    
public:
    vector<string> words = {"cat", "tac", "dog", "god", "cow"};
    
    // Constructor - precompute on initialization
    Solution() {
        precompute();
    }
    
    // Given (hidden implementation)
    string getHash(string word);
    
    // Score function
    int getScore(const string& word) {
        int score = 0;
        for (int i = 1; i < word.size(); i++) {
            score += abs(word[i] - word[i - 1]);
        }
        return score;
    }
    
    // Optimized query - O(1) after precomputation
    string findAnagram(string word) {
        string h = getHash(word);
        
        // If no anagram exists for this hash
        if (bestAnagram.find(h) == bestAnagram.end()) {
            return "";
        }
        
        string best = bestAnagram[h];
        
        // If best word is the input word itself, we need to find another one
        if (best == word) {
            // We need to check all anagrams for this hash to find the second best
            // But wait - we only stored the best. Let's handle this differently.
            // Actually, we should store all anagrams and their scores, then pick best != word
            return "";  // Or we need to store more info
        }
        
        return best;
    }
};
```

### Fully Optimized Solution (Storing All Anagrams)

Since we need to skip the input word, we should store all anagrams sorted by score:

```cpp
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
using namespace std;

// Comparator function for sorting by score (descending)
bool cmp(const pair<string, int>& a, const pair<string, int>& b) {
    return a.second > b.second;  // Higher score first
}

class Solution {
private:
    // Precomputed data: hash -> sorted list of (word, score) pairs
    unordered_map<string, vector<pair<string, int>>> anagramMap;
    
    // Precomputation method
    void precompute() {
        // Map: hash -> vector of (word, score) pairs
        unordered_map<string, vector<pair<string, int>>> info;
        
        // 1. Group words by hash and calculate scores
        for (const string& w : words) {
            string h = getHash(w);
            int s = getScore(w);
            
            // Store word and its score for this hash
            info[h].push_back({w, s});
        }
        
        // 2. For each hash group, sort by score and store
        for (auto& [hash, vec] : info) {
            // Sort by score (descending) - highest score first
            sort(vec.begin(), vec.end(), cmp);
            
            // Store the sorted list for this hash
            anagramMap[hash] = vec;
        }
    }
    
public:
    vector<string> words = {"cat", "tac", "dog", "god", "cow"};
    
    // Constructor - precompute on initialization
    Solution() {
        precompute();
    }
    
    // Given (hidden implementation)
    string getHash(string word);
    
    // Score function: sum of absolute differences between adjacent characters
    int getScore(const string& word) {
        int score = 0;
        for (int i = 1; i < word.size(); i++) {
            score += abs(word[i] - word[i - 1]);
        }
        return score;
    }
    
    // Optimized query - O(k) where k is number of anagrams (usually small)
    string findAnagram(string word) {
        string h = getHash(word);
        
        // If no anagrams exist for this hash
        if (anagramMap.find(h) == anagramMap.end()) {
            return "";
        }
        
        // Get sorted list of anagrams for this hash
        const vector<pair<string, int>>& anagrams = anagramMap[h];
        
        // Find the best scoring word that's not the input word
        for (const auto& [w, score] : anagrams) {
            if (w != word) {
                return w;  // Return first (best scoring) that's not the input
            }
        }
        
        // No anagram found (input was the only one)
        return "";
    }
};
```

### Explanation

**Precomputation Phase:**
- **Line 20-35**: `precompute()` function:
  - **Line 22-30**: For each word, calculate hash and score, group by hash in `info` map
  - **Line 32-37**: For each hash group, sort by score (descending) using comparator `cmp`
  - **Line 39**: Store sorted list in `anagramMap` for O(1) lookup during queries

**Query Phase:**
- **Line 58-75**: `findAnagram()` function:
  - **Line 60**: Get hash of input word
  - **Line 62-65**: Check if anagrams exist for this hash
  - **Line 67-68**: Get sorted list of anagrams (already sorted by score, highest first)
  - **Line 70-74**: Iterate through sorted list, return first word that's not the input
  - Since list is sorted by score, first non-input word is the best scoring anagram

**Key Optimizations:**
1. **Precomputation**: Calculate hash and score once per word during initialization
2. **Grouping**: Group anagrams by hash (O(1) lookup)
3. **Sorting**: Sort once during precomputation, not per query
4. **Early Return**: Return first valid anagram (best scoring, not input)

## Complexity Analysis

### Time Complexity

**Precomputation:**
- Calculate hash for N words: O(N) [assuming getHash is O(1)]
- Calculate score for N words: O(N × len(word))
- Sorting: O(k × log k) for each hash group with k anagrams
- **Overall**: O(N × len(word) + sorting overhead)

**Query:**
- Hash lookup: O(1)
- Finding best anagram != word: O(k) where k is number of anagrams (usually small, worst case N)
- **Overall**: O(k) per query, which is O(1) in practice if k is small

### Space Complexity
- **O(N)**: Store all words and scores in the map
- **Overall**: O(N)

### Comparison

| Approach | Precomputation | Query Time | Space |
|----------|---------------|------------|-------|
| Brute Force | None | O(N × len(word)) | O(1) |
| Optimized | O(N × len(word)) | O(k) ≈ O(1) | O(N) |

**When to use each:**
- **Brute Force**: Few queries, small dictionary
- **Optimized**: Many queries, large dictionary (better amortized cost)

## Key Insights
- **Precomputation Trade-off**: Spend time upfront to make queries fast
- **Hash Grouping**: All anagrams share the same hash - use this for efficient grouping
- **Sorting Once**: Sort during precomputation, not per query
- **Skip Input Word**: Since we need a different word, we iterate through sorted list until we find one that's not the input
- **Comparator Function**: Using external comparator is cleaner and more readable than lambda

## Alternative Approaches

### Using Priority Queue (Not Recommended Here)

While we could use a priority queue, it's not optimal because:
- We need to store all anagrams anyway (to skip input word)
- Sorting once is simpler than maintaining a heap
- Priority queue doesn't help when we need to skip a specific element

However, if we only needed the best (and input was guaranteed to not be the best), we could use:

```cpp
// Not optimal for this problem, but shown for completeness
unordered_map<string, priority_queue<pair<int, string>>> anagramPQ;

// During precomputation
anagramPQ[h].push({score, word});  // Max-heap by default (score first)

// During query
auto& pq = anagramPQ[h];
while (!pq.empty() && pq.top().second == word) {
    pq.pop();  // Skip input word
}
return pq.empty() ? "" : pq.top().second;
```

**Why not optimal:**
- We'd need to pop elements to skip input word (destructive)
- Sorting is simpler and non-destructive
- We can reuse sorted list for multiple queries

## Related Problems
- Group Anagrams
- Find All Anagrams in a String
- Valid Anagram
- Anagram Mappings


