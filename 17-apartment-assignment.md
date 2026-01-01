# Apartment Assignment to People

## Problem Statement

Your organization has hired interns who need to relocate for the summer. You are in charge of assigning apartments to them. Each intern will get their own room. They can choose whether they prefer to share a 2+ room apartment or get a one-bedroom to themselves.

**Note:** They may not get what they want because the apartments vary in the number of rooms they have.

**Data Structures:**
```cpp
struct Apartment {
    int aptNumber;
    int numRooms;
};

struct Person {
    string name;           // Unique ID
    bool wants_housemates; // true = wants to share, false = wants solo
};
```

**Constraints:**
- Each person gets exactly one room
- No apartment can exceed its room capacity
- People who don't want housemates (`wants_housemates == false`) must be alone in their apartment
- People who want housemates (`wants_housemates == true`) can share apartments

**Input Format:**
- `apartments`: Vector of apartments with number and room count
- `people`: Vector of people with name and preference

**Output:** Map from apartment number to list of people assigned to that apartment.

## Examples

### Example 1
```
Input:
apartments = [
    {aptNumber: 1, numRooms: 1},
    {aptNumber: 2, numRooms: 2},
    {aptNumber: 3, numRooms: 3}
]

people = [
    {name: "Alice", wants_housemates: false},
    {name: "Bob", wants_housemates: true},
    {name: "Charlie", wants_housemates: true},
    {name: "David", wants_housemates: false}
]

Assignment:
- Step 1: Alice (no housemates) → Apt 1 (single-room)
- Step 2: David (no housemates) → Apt 2 (multi-room, but locked)
- Step 3: Bob, Charlie (want housemates) → Apt 3 (2 rooms available)

Output:
{
    1: ["Alice"],
    2: ["David"],
    3: ["Bob", "Charlie"]
}
```

### Example 2
```
Input:
apartments = [
    {aptNumber: 1, numRooms: 1},
    {aptNumber: 2, numRooms: 1},
    {aptNumber: 3, numRooms: 2}
]

people = [
    {name: "Eve", wants_housemates: false},
    {name: "Frank", wants_housemates: false},
    {name: "Grace", wants_housemates: false},
    {name: "Henry", wants_housemates: true}
]

Assignment:
- Step 1: Eve → Apt 1, Frank → Apt 2 (both single-room)
- Step 2: Grace → Apt 3 (multi-room, locked)
- Step 3: Henry → (no space left)

Output:
{
    1: ["Eve"],
    2: ["Frank"],
    3: ["Grace"]
}
```

### Example 3
```
Input:
apartments = [
    {aptNumber: 1, numRooms: 3},
    {aptNumber: 2, numRooms: 2}
]

people = [
    {name: "Ivy", wants_housemates: true},
    {name: "Jack", wants_housemates: true},
    {name: "Kate", wants_housemates: true},
    {name: "Leo", wants_housemates: true}
]

Assignment:
- Step 1: No non-flexible people
- Step 2: No non-flexible people
- Step 3: Ivy, Jack → Apt 1 (2 rooms), Kate, Leo → Apt 2 (2 rooms)

Output:
{
    1: ["Ivy", "Jack"],
    2: ["Kate", "Leo"]
}
```

## Constraints
- `1 <= apartments.length <= 10^4`
- `1 <= people.length <= 10^4`
- `1 <= numRooms <= 10`
- Each person name is unique
- Each apartment number is unique

## Approach

### Step 1: Understanding the Problem
This is an **allocation problem with capacity constraints and soft preferences**.

**Key Constraints:**
- Hard constraint: No apartment exceeds room capacity
- Hard constraint: People who don't want housemates must be alone
- Soft preference: People who want housemates prefer sharing (but can be alone if needed)

**Goal:** Assign people to apartments respecting hard constraints and satisfying preferences when possible.

### Step 2: Identifying the Strategy
**Greedy Principle:** Assign the **most constrained people first**.

**Categorization:**
- **Apartments:**
  - Single-room apartments (`numRooms == 1`)
  - Multi-room apartments (`numRooms > 1`)
  
- **People:**
  - Non-flexible (`wants_housemates == false`) - must be alone
  - Flexible (`wants_housemates == true`) - can adapt

**Why This Order?**
- Non-flexible people restrict choices (must be alone)
- Flexible people can fill any remaining space
- Handle constraints first, then optimize preferences

### Step 3: Algorithm Design
1. **Categorize inputs**: Separate apartments and people by type
2. **Step 1**: Assign non-flexible people to single-room apartments (best match)
3. **Step 2**: Assign remaining non-flexible people to multi-room apartments (lock them)
4. **Step 3**: Assign flexible people to any remaining capacity
5. **Track capacity**: Use map to track remaining rooms per apartment

**Invariant:** At all times:
- No apartment exceeds capacity
- No non-flexible person ever gets a housemate
- Flexible people fill only valid leftover space

### Step 4: Edge Cases
- More non-flexible people than single-room apartments: Use multi-room apartments
- More people than total rooms: Some people won't be assigned
- All people flexible: Fill apartments greedily
- All people non-flexible: One per apartment, may not fit all

## Solution

### Code Implementation

```cpp
#include <vector>
#include <map>
#include <queue>
#include <unordered_map>
#include <string>
using namespace std;

struct Apartment {
    int aptNumber;
    int numRooms;
};

struct Person {
    string name;
    bool wants_housemates;
};

class Solution {
public:
    /**
     * Assign people to apartments respecting preferences and constraints
     * 
     * @param apartments List of available apartments
     * @param people List of people to assign
     * @return Map from apartment number to list of assigned people
     */
    map<int, vector<string>> assignApartmentsToPeople(
        vector<Apartment>& apartments,
        vector<Person>& people
    ) {
        // Result: apartment number -> list of people assigned
        map<int, vector<string>> result;
        
        // ---------- Separate apartments ----------
        vector<Apartment> singleRoomApts;
        vector<Apartment> multiRoomApts;
        
        for (auto& apt : apartments) {
            if (apt.numRooms == 1) {
                singleRoomApts.push_back(apt);
            } else {
                multiRoomApts.push_back(apt);
            }
        }
        
        // ---------- Separate people by flexibility ----------
        queue<Person> noHousemates;     // wants_housemates == false
        queue<Person> wantsHousemates;  // wants_housemates == true
        
        for (auto& p : people) {
            if (!p.wants_housemates) {
                noHousemates.push(p);
            } else {
                wantsHousemates.push(p);
            }
        }
        
        // ---------- Track remaining capacity ----------
        // remainingRooms[aptNumber] = how many rooms are still usable
        unordered_map<int, int> remainingRooms;
        for (auto& apt : apartments) {
            remainingRooms[apt.aptNumber] = apt.numRooms;
        }
        
        // ---------- STEP 1: Assign non-flexible people to single-room apartments ----------
        // Best match: satisfies preference perfectly
        for (auto& apt : singleRoomApts) {
            if (!noHousemates.empty()) {
                result[apt.aptNumber].push_back(noHousemates.front().name);
                noHousemates.pop();
                
                // Single-room apartment is now full
                remainingRooms[apt.aptNumber] = 0;
            }
        }
        
        // ---------- STEP 2: Assign remaining non-flexible people to multi-room apartments ----------
        // IMPORTANT: Even though these apartments have multiple rooms,
        // we must keep them exclusive (only one person),
        // so we force remainingRooms = 0.
        for (auto& apt : multiRoomApts) {
            if (!noHousemates.empty()) {
                result[apt.aptNumber].push_back(noHousemates.front().name);
                noHousemates.pop();
                
                // Lock the apartment so no one else can be added
                remainingRooms[apt.aptNumber] = 0;
            }
        }
        
        // ---------- STEP 3: Assign flexible people ----------
        // Flexible people can go anywhere with remaining capacity
        for (auto& apt : apartments) {
            int aptNo = apt.aptNumber;
            
            while (remainingRooms[aptNo] > 0 && !wantsHousemates.empty()) {
                result[aptNo].push_back(wantsHousemates.front().name);
                wantsHousemates.pop();
                remainingRooms[aptNo]--;
            }
        }
        
        return result;
    }
};
```

### Explanation
- **Line 30-37**: Categorize apartments:
  - **singleRoomApts**: Apartments with exactly 1 room
  - **multiRoomApts**: Apartments with 2+ rooms
  
- **Line 39-47**: Categorize people:
  - **noHousemates**: People who don't want housemates (non-flexible)
  - **wantsHousemates**: People who want housemates (flexible)
  
- **Line 49-54**: Track remaining capacity:
  - **remainingRooms**: Map from apartment number to available rooms
  
- **Line 56-65**: **Step 1 - Assign non-flexible to single-room**:
  - Best match: satisfies preference perfectly
  - Assign one person per single-room apartment
  - Mark apartment as full (remainingRooms = 0)
  
- **Line 67-78**: **Step 2 - Assign remaining non-flexible to multi-room**:
  - Assign one person per multi-room apartment
  - **Lock apartment** (remainingRooms = 0) to prevent others from joining
  - Ensures non-flexible person stays alone
  
- **Line 80-89**: **Step 3 - Assign flexible people**:
  - Fill any apartment with remaining capacity
  - Greedily assign until rooms or people run out
  - Flexible people can share or be alone

**Why This Works:**
- **Hard constraints respected**: Non-flexible people are always alone
- **Preferences satisfied when possible**: Non-flexible get single-room first
- **Greedy is correct**: Non-flexible people restrict choices, flexible people adapt
- **Capacity never exceeded**: Tracked via remainingRooms map

## Complexity Analysis

### Time Complexity
- **O(A + P)**: 
  - `A` = number of apartments
  - `P` = number of people
  - Categorization: O(A + P)
  - Assignment: O(A + P) in worst case
  - **Overall**: O(A + P)

### Space Complexity
- **O(A + P)**: 
  - Result map: O(A + P) in worst case
  - Queues: O(P)
  - remainingRooms map: O(A)
  - **Overall**: O(A + P)

## Key Insights
- **Greedy Strategy**: Handle most constrained people first
- **Categorization**: Separate by flexibility and capacity
- **Locking Mechanism**: Multi-room apartments assigned to non-flexible people are locked
- **Three-Step Process**: Single-room → Multi-room (locked) → Flexible fill
- **Invariant**: Hard constraints always respected, preferences satisfied when possible
- **Queue Usage**: Process people in order, maintain flexibility separation

## Alternative Approaches

### Using Priority Queue (If Preferences Have Priorities)

```cpp
map<int, vector<string>> assignApartmentsWithPriority(
    vector<Apartment>& apartments,
    vector<Person>& people
) {
    map<int, vector<string>> result;
    
    // Separate by flexibility
    vector<Person> noHousemates, wantsHousemates;
    for (auto& p : people) {
        if (!p.wants_housemates) {
            noHousemates.push_back(p);
        } else {
            wantsHousemates.push_back(p);
        }
    }
    
    // Sort apartments by room count (ascending) for better matching
    sort(apartments.begin(), apartments.end(), 
         [](const Apartment& a, const Apartment& b) {
             return a.numRooms < b.numRooms;
         });
    
    unordered_map<int, int> remainingRooms;
    for (auto& apt : apartments) {
        remainingRooms[apt.aptNumber] = apt.numRooms;
    }
    
    // Same three-step process...
    // (Implementation similar to main solution)
    
    return result;
}
```

**Time Complexity:** O(A log A + P)  
**Space Complexity:** O(A + P)

**When to Use:** If apartments need to be processed in a specific order (e.g., smallest first)

### Using Bipartite Matching (For Optimal Assignment)

```cpp
// More complex approach using maximum bipartite matching
// Only needed if we want to maximize number of satisfied preferences
// Overkill for this problem since greedy works perfectly
```

**Time Complexity:** O(A × P) or more  
**Space Complexity:** O(A × P)

**When to Use:** If we need to maximize number of satisfied preferences (not just feasible assignment)

## Related Problems
- Meeting Rooms II
- Task Scheduler
- Course Schedule
- Maximum Bipartite Matching

