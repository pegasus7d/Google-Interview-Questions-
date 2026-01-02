# File System Consolidation

## Problem Statement

Given a complete list of all files in a file system and a subset of selected files, return a consolidated list where:

- If **all files in a directory** are in the subset, replace those file names with the **directory name**
- Otherwise, keep the individual selected files

**Rules:**
1. Process directories independently (each directory is checked separately)
2. For each directory, check if all its direct children (files) are selected
3. If yes, replace all those files with the directory name
4. If no, keep only the selected files from that directory

**Input Format:**
- `allFiles`: Vector of all file paths in the system (e.g., `"a/b/c/d.txt"`)
- `subsetFiles`: Vector of selected file paths

**Output:** Consolidated list where directories replace all their files when all files are selected.

## Examples

### Example 1
```
Input:
allFiles = [
    "a/b/c/d.txt",
    "a/b/c/e.txt",
    "a/b/b.txt",
    "a/b/e.txt",
    "b/c/d.txt"
]

subsetFiles = [
    "a/b/c/d.txt",
    "a/b/c/e.txt",
    "a/b/b.txt",
    "b/c/d.txt"
]

Processing by directory:
- Directory "a/b/c": 
  - Files: ["a/b/c/d.txt", "a/b/c/e.txt"]
  - All selected? Yes → Consolidate to "a/b/c"
  
- Directory "a/b":
  - Files: ["a/b/b.txt", "a/b/e.txt"]
  - All selected? No (e.txt not selected) → Keep "a/b/b.txt"
  
- Directory "b/c":
  - Files: ["b/c/d.txt"]
  - All selected? Yes → Consolidate to "b/c"

Output: ["a/b/c", "a/b/b.txt", "b/c"]
```

### Example 2
```
Input:
allFiles = [
    "root/file1.txt",
    "root/file2.txt",
    "root/file3.txt",
    "root/sub/file4.txt"
]

subsetFiles = [
    "root/file1.txt",
    "root/file2.txt",
    "root/file3.txt"
]

Processing:
- Directory "root":
  - Files: ["root/file1.txt", "root/file2.txt", "root/file3.txt"]
  - All selected? Yes → Consolidate to "root"
  
- Directory "root/sub":
  - Files: ["root/sub/file4.txt"]
  - All selected? No → (nothing to add)

Output: ["root"]
```

### Example 3
```
Input:
allFiles = [
    "x/y/a.txt",
    "x/y/b.txt",
    "x/z/c.txt"
]

subsetFiles = [
    "x/y/a.txt",
    "x/z/c.txt"
]

Processing:
- Directory "x/y":
  - Files: ["x/y/a.txt", "x/y/b.txt"]
  - All selected? No → Keep "x/y/a.txt"
  
- Directory "x/z":
  - Files: ["x/z/c.txt"]
  - All selected? Yes → Consolidate to "x/z"

Output: ["x/y/a.txt", "x/z"]
```

## Constraints
- `1 <= allFiles.length <= 10^4`
- `1 <= subsetFiles.length <= 10^4`
- All file paths are valid (non-empty, properly formatted)
- File paths use `/` as separator
- All files in `subsetFiles` are present in `allFiles`
- Paths are relative (no leading `/`)

## Approach

### Step 1: Understanding the Problem
This is a **directory-based consolidation** problem:
- Group files by their **parent directory**
- For each directory, check if **all its files** are selected
- If yes, replace files with directory name
- If no, keep only selected files

**Key Insight:** Process directories independently - each directory's consolidation decision is independent of others.

### Step 2: Identifying the Strategy
**Grouping and Checking:**
1. Create a set of selected files for fast lookup
2. Group all files by their parent directory
3. For each directory:
   - Check if all files in that directory are selected
   - If yes, add directory to result
   - If no, add only selected files to result

**Data Structures:**
- `unordered_set<string>`: Fast lookup of selected files
- `unordered_map<string, vector<string>>`: Map directory → files in that directory

### Step 3: Algorithm Design
1. **Build selected set**: Convert `subsetFiles` to set for O(1) lookup
2. **Group by directory**: For each file in `allFiles`, add to its parent directory's list
3. **Process each directory**:
   - Get all files in directory
   - Check if all files are selected
   - If yes: Add directory to result
   - If no: Add only selected files to result
4. **Return consolidated list**

### Step 4: Edge Cases
- Empty subset: Return empty list
- All files selected: May consolidate to root directory
- Single file in directory: Consolidate if selected
- Nested directories: Process each level independently
- Root directory files: Handle empty parent directory

## Solution

### Code Implementation

```cpp
#include <vector>
#include <string>
#include <unordered_set>
#include <unordered_map>
using namespace std;

class Solution {
private:
    /**
     * Get parent directory of a file path
     * 
     * @param path File path (e.g., "a/b/c/d.txt")
     * @return Parent directory (e.g., "a/b/c")
     */
    string getParentDirectory(const string& path) {
        int pos = path.find_last_of('/');
        if (pos == string::npos) {
            return "";  // Root level file
        }
        return path.substr(0, pos);
    }
    
public:
    /**
     * Consolidate files: replace files with directory if all files in directory are selected
     * 
     * @param allFiles Complete list of all files in file system
     * @param subsetFiles Subset of selected files
     * @return Consolidated list (directories replace files when all files selected)
     */
    vector<string> consolidateFiles(
        const vector<string>& allFiles,
        const vector<string>& subsetFiles
    ) {
        // Fast lookup of selected files
        unordered_set<string> selected(
            subsetFiles.begin(), 
            subsetFiles.end()
        );
        
        // Map: directory -> all files directly inside it
        unordered_map<string, vector<string>> dirToFiles;
        
        for (const string& file : allFiles) {
            string parent = getParentDirectory(file);
            dirToFiles[parent].push_back(file);
        }
        
        vector<string> result;
        
        // Process each directory independently
        for (const auto& entry : dirToFiles) {
            const string& dir = entry.first;
            const vector<string>& files = entry.second;
            
            // Check if all files in this directory are selected
            bool allSelected = true;
            for (const string& f : files) {
                if (!selected.count(f)) {
                    allSelected = false;
                    break;
                }
            }
            
            if (allSelected) {
                // All files selected → collapse directory
                result.push_back(dir);
            } else {
                // Not all files selected → keep only selected files
                for (const string& f : files) {
                    if (selected.count(f)) {
                        result.push_back(f);
                    }
                }
            }
        }
        
        return result;
    }
};
```

### Explanation
- **Line 15-24**: `getParentDirectory` helper function:
  - **Line 19**: Find last `/` in path
  - **Line 20-22**: Handle root level files (no `/`)
  - **Line 23**: Return substring before last `/`
  
- **Line 30-75**: `consolidateFiles` function:
  - **Line 33-36**: **Build selected set**:
    - Convert `subsetFiles` to `unordered_set` for O(1) lookup
  - **Line 38-44**: **Group files by directory**:
    - For each file, get its parent directory
    - Add file to that directory's list in `dirToFiles` map
  - **Line 46**: Initialize result vector
  - **Line 48-70**: **Process each directory**:
    - **Line 52-58**: Check if all files in directory are selected:
      - Iterate through all files in directory
      - If any file not in selected set, set `allSelected = false`
    - **Line 60-63**: **If all selected**:
      - Add directory to result (consolidate)
    - **Line 64-69**: **If not all selected**:
      - Add only selected files to result
  - **Line 72**: Return consolidated list

**Why This Works:**
- **Independent Processing**: Each directory is processed independently
- **Fast Lookup**: Set provides O(1) membership check
- **Complete Check**: Verify all files in directory before consolidating
- **Selective Addition**: Only add selected files when not consolidating

**Example Walkthrough:**
```
allFiles = ["a/b/c/d.txt", "a/b/c/e.txt", "a/b/b.txt", "a/b/e.txt", "b/c/d.txt"]
subsetFiles = ["a/b/c/d.txt", "a/b/c/e.txt", "a/b/b.txt", "b/c/d.txt"]

Step 1: Build selected set
  selected = {"a/b/c/d.txt", "a/b/c/e.txt", "a/b/b.txt", "b/c/d.txt"}

Step 2: Group by directory
  dirToFiles = {
    "a/b/c": ["a/b/c/d.txt", "a/b/c/e.txt"],
    "a/b": ["a/b/b.txt", "a/b/e.txt"],
    "b/c": ["b/c/d.txt"]
  }

Step 3: Process each directory
  Directory "a/b/c":
    Files: ["a/b/c/d.txt", "a/b/c/e.txt"]
    All selected? Yes → Add "a/b/c"
  
  Directory "a/b":
    Files: ["a/b/b.txt", "a/b/e.txt"]
    All selected? No (e.txt missing) → Add "a/b/b.txt"
  
  Directory "b/c":
    Files: ["b/c/d.txt"]
    All selected? Yes → Add "b/c"

Result: ["a/b/c", "a/b/b.txt", "b/c"]
```

## Complexity Analysis

### Time Complexity
- **O(N + M)**: 
  - `N` = number of files in `allFiles`
  - `M` = number of files in `subsetFiles`
  - Building selected set: O(M)
  - Grouping files by directory: O(N)
  - Processing directories: O(N) in worst case
  - **Overall**: O(N + M)

### Space Complexity
- **O(N + M)**: 
  - Selected set: O(M)
  - Directory map: O(N) in worst case
  - Result vector: O(N) in worst case
  - **Overall**: O(N + M)

## Key Insights
- **Directory-Based Grouping**: Group files by parent directory to check completeness
- **Independent Processing**: Each directory's consolidation is independent
- **Fast Lookup**: Use set for O(1) membership checking
- **Complete Check**: Must verify ALL files in directory are selected before consolidating
- **Selective Addition**: When not consolidating, add only selected files
- **Parent Extraction**: Extract parent directory using `find_last_of('/')`

## Alternative Approaches

### Using Trie (For Hierarchical Processing)

```cpp
struct TrieNode {
    unordered_map<string, TrieNode*> children;
    vector<string> files;
    bool isSelected;
};

vector<string> consolidateFilesTrie(
    const vector<string>& allFiles,
    const vector<string>& subsetFiles
) {
    unordered_set<string> selected(subsetFiles.begin(), subsetFiles.end());
    
    // Build trie structure
    TrieNode* root = new TrieNode();
    
    for (const string& file : allFiles) {
        // Parse path and build trie
        // More complex but allows hierarchical processing
    }
    
    // Process trie to consolidate
    // ...
}
```

**Time Complexity:** O(N × L) where L = average path length  
**Space Complexity:** O(N × L)

**When to Use:**
- Need hierarchical processing (parent-child relationships)
- Want to consolidate at multiple levels simultaneously
- More complex but more flexible

### Sorting-Based Approach

```cpp
vector<string> consolidateFilesSorted(
    const vector<string>& allFiles,
    const vector<string>& subsetFiles
) {
    unordered_set<string> selected(subsetFiles.begin(), subsetFiles.end());
    
    // Sort files by directory
    map<string, vector<string>> dirToFiles;
    for (const string& file : allFiles) {
        string parent = getParentDirectory(file);
        dirToFiles[parent].push_back(file);
    }
    
    // Process in sorted order (ensures consistent output)
    vector<string> result;
    for (const auto& entry : dirToFiles) {
        // Same logic as main solution
    }
    
    return result;
}
```

**Time Complexity:** O(N log N + M)  
**Space Complexity:** O(N + M)

**Advantage:** Consistent output order (sorted by directory)

## Related Problems
- File System Traversal
- Directory Tree Operations
- Path Manipulation
- Group Anagrams (similar grouping concept)

