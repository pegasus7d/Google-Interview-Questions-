# First Expired RPC

## Problem Statement

We have logs that print when RPC (Remote Procedure Call) requests come in and go out. Each log entry contains:
- `RPC_ID`: Unique identifier for the RPC
- `Timestamp`: Time when the event occurred
- `Type`: Either `"in"` (RPC started) or `"out"` (RPC completed)

Additionally, we are given a **timeout** value. An RPC expires if:
- It came in at time `T`
- It has not completed (no "out" log) by time `T + timeout`
- OR it completed after `T + timeout` (completed too late)

Find the **first time** when any RPC expires.

**Input Format:**
- `n`: Number of log entries
- `timeout`: Timeout duration
- `logs`: Array of log entries, each with `{RPC_ID, Timestamp, Type}`

**Output:** The first timestamp when any RPC expires, or `-1` if no RPC expires.

## Examples

### Example 1
```
Input:
n = 6, timeout = 5
logs = [
    {RPC1, 10, "in"},
    {RPC2, 12, "in"},
    {RPC1, 13, "out"},
    {RPC3, 15, "in"},
    {RPC2, 20, "out"},
    {RPC3, 25, "out"}
]

RPC Analysis:
- RPC1: in at 10, out at 13
  - Expires at: 10 + 5 = 15
  - Completed at 13 < 15 ✓ (not expired)
  
- RPC2: in at 12, out at 20
  - Expires at: 12 + 5 = 17
  - Completed at 20 > 17 ✗ (expired at time 17)
  
- RPC3: in at 15, out at 25
  - Expires at: 15 + 5 = 20
  - Completed at 25 > 20 ✗ (expired at time 20)

Earliest expiration: min(17, 20) = 17
Output: 17
```

### Example 2
```
Input:
n = 4, timeout = 3
logs = [
    {RPC1, 5, "in"},
    {RPC2, 7, "in"},
    {RPC1, 8, "out"},
    {RPC2, 9, "out"}
]

RPC Analysis:
- RPC1: in at 5, out at 8
  - Expires at: 5 + 3 = 8
  - Completed at 8 = 8 ✓ (completed exactly at expiration, not expired)
  
- RPC2: in at 7, out at 9
  - Expires at: 7 + 3 = 10
  - Completed at 9 < 10 ✓ (not expired)

No RPC expired
Output: -1
```

### Example 3
```
Input:
n = 3, timeout = 10
logs = [
    {RPC1, 0, "in"},
    {RPC2, 5, "in"},
    {RPC1, 15, "out"}
]

RPC Analysis:
- RPC1: in at 0, out at 15
  - Expires at: 0 + 10 = 10
  - Completed at 15 > 10 ✗ (expired at time 10)
  
- RPC2: in at 5, no "out" log
  - Expires at: 5 + 10 = 15
  - Never completed ✗ (expired at time 15)

Earliest expiration: min(10, 15) = 10
Output: 10
```

## Constraints
- `1 <= n <= 10^5`
- `1 <= timeout <= 10^9`
- `1 <= Timestamp <= 10^9`
- All RPC_IDs are strings
- Type is either "in" or "out"
- An RPC may have only "in" log (never completed)
- An RPC may have only "out" log (started before logs began)

## Approach

### Step 1: Understanding the Problem
An RPC expires if:
1. It started at time `T_in`
2. It expires at time `T_in + timeout`
3. Either:
   - No "out" log exists (RPC never completed)
   - "out" log exists but `T_out > T_in + timeout` (completed too late)

We need to find the **earliest expiration time** among all expired RPCs.

### Step 2: Identifying the Strategy
1. **Parse logs**: Separate "in" and "out" logs by RPC_ID
2. **Track timestamps**: Store in-time and out-time for each RPC
3. **Check expiration**: For each RPC with "in" log:
   - Calculate expiration time: `inTime + timeout`
   - Check if expired: no "out" OR `outTime > expirationTime`
4. **Find minimum**: Return the earliest expiration time

### Step 3: Algorithm Design
1. **Create two maps**:
   - `inTime[rpcId]`: Timestamp when RPC started
   - `outTime[rpcId]`: Timestamp when RPC completed
2. **Process all logs**: Populate both maps
3. **Check each RPC**:
   - For each RPC in `inTime`:
     - Calculate `expireAt = inTime[rpcId] + timeout`
     - If `outTime` doesn't exist OR `outTime[rpcId] > expireAt`:
       - RPC expired at `expireAt`
       - Track minimum expiration time
4. **Return result**: Minimum expiration time, or -1 if none expired

### Step 4: Edge Cases
- RPC with only "in" log: Expires at `inTime + timeout`
- RPC with only "out" log: Ignore (no start time, can't determine expiration)
- RPC completed exactly at expiration: Not expired (completed in time)
- Multiple RPCs expire at same time: Return that time
- No RPCs expire: Return -1

## Solution

### Code Implementation

```cpp
#include <vector>
#include <string>
#include <unordered_map>
#include <climits>
#include <iostream>
using namespace std;

struct Log {
    string rpcId;
    int timestamp;
    string type;  // "in" or "out"
};

/**
 * Find the first time when any RPC expires
 * 
 * @param logs Vector of log entries
 * @param timeout Timeout duration
 * @return First expiration timestamp, or -1 if no RPC expires
 */
int firstExpiredRPC(vector<Log>& logs, int timeout) {
    // Maps to store in-time and out-time for each RPC
    unordered_map<string, int> inTime;
    unordered_map<string, int> outTime;
    
    // Process all logs and store timestamps
    for (const auto& log : logs) {
        if (log.type == "in") {
            inTime[log.rpcId] = log.timestamp;
        } else {  // "out"
            outTime[log.rpcId] = log.timestamp;
        }
    }
    
    int earliestExpiration = INT_MAX;
    bool found = false;
    
    // Check each RPC that has an "in" log
    for (const auto& entry : inTime) {
        const string& rpcId = entry.first;
        int inTs = entry.second;
        int expireAt = inTs + timeout;
        
        // RPC expires if:
        // 1. No "out" log exists (never completed), OR
        // 2. "out" timestamp is greater than expiration time (completed too late)
        if (!outTime.count(rpcId) || outTime[rpcId] > expireAt) {
            earliestExpiration = min(earliestExpiration, expireAt);
            found = true;
        }
    }
    
    return found ? earliestExpiration : -1;
}

int main() {
    int n, timeout;
    cin >> n >> timeout;
    
    vector<Log> logs(n);
    
    for (int i = 0; i < n; i++) {
        cin >> logs[i].rpcId >> logs[i].timestamp >> logs[i].type;
    }
    
    int result = firstExpiredRPC(logs, timeout);
    
    if (result == -1) {
        cout << "No RPC expired\n";
    } else {
        cout << "First RPC expires at time: " << result << "\n";
    }
    
    return 0;
}
```

### Explanation
- **Line 18-25**: Define Log structure with RPC_ID, timestamp, and type
- **Line 27-55**: `firstExpiredRPC` function:
  - **Line 30-31**: Create maps to store in-time and out-time for each RPC
  - **Line 33-40**: Process all logs:
    - **Line 35-37**: If "in" log, store timestamp in `inTime` map
    - **Line 38-39**: If "out" log, store timestamp in `outTime` map
  - **Line 42-43**: Initialize variables to track earliest expiration
  - **Line 45-54**: Check each RPC that started:
    - **Line 47-48**: Get RPC ID and in-time
    - **Line 49**: Calculate expiration time: `inTime + timeout`
    - **Line 51-54**: Check if RPC expired:
      - No "out" log: `!outTime.count(rpcId)` → expired
      - Completed too late: `outTime[rpcId] > expireAt` → expired
    - **Line 52-53**: Update earliest expiration time
  - **Line 56**: Return earliest expiration or -1 if none found

**Key Logic:**
- An RPC expires at `inTime + timeout` if it hasn't completed by then
- We only check RPCs that have an "in" log (we know when they started)
- If an RPC has no "out" log, it never completed → expired
- If an RPC completed after expiration time, it expired

## Complexity Analysis

### Time Complexity
- **O(n)**: 
  - Process all logs once: O(n)
  - Iterate through `inTime` map: O(n) in worst case
  - Each operation (map lookup, insertion) is O(1) average
  - **Overall**: O(n)

### Space Complexity
- **O(n)**: 
  - `inTime` map: O(n) in worst case (all unique RPCs)
  - `outTime` map: O(n) in worst case
  - **Overall**: O(n)

## Key Insights
- **Two-Phase Processing**: First collect all logs, then analyze expiration
- **Map-Based Lookup**: Use hash maps for O(1) lookup of RPC timestamps
- **Expiration Logic**: RPC expires if no completion OR completion too late
- **Edge Case Handling**: RPCs with only "out" logs are ignored (no start time)
- **Exact Boundary**: If RPC completes exactly at expiration time, it's not expired (completed in time)

## Alternative Approaches

### Single Pass with Early Termination

```cpp
int firstExpiredRPCSinglePass(vector<Log>& logs, int timeout) {
    unordered_map<string, int> inTime;
    unordered_map<string, int> outTime;
    int earliestExpiration = INT_MAX;
    bool found = false;
    
    // Process logs and check expiration on the fly
    for (const auto& log : logs) {
        if (log.type == "in") {
            inTime[log.rpcId] = log.timestamp;
            // Check if this RPC already expired
            int expireAt = log.timestamp + timeout;
            if (outTime.count(log.rpcId) && outTime[log.rpcId] > expireAt) {
                earliestExpiration = min(earliestExpiration, expireAt);
                found = true;
            }
        } else {  // "out"
            outTime[log.rpcId] = log.timestamp;
            // Check if this RPC expired
            if (inTime.count(log.rpcId)) {
                int expireAt = inTime[log.rpcId] + timeout;
                if (log.timestamp > expireAt) {
                    earliestExpiration = min(earliestExpiration, expireAt);
                    found = true;
                }
            }
        }
    }
    
    // Check RPCs that never completed
    for (const auto& entry : inTime) {
        if (!outTime.count(entry.first)) {
            int expireAt = entry.second + timeout;
            earliestExpiration = min(earliestExpiration, expireAt);
            found = true;
        }
    }
    
    return found ? earliestExpiration : -1;
}
```

**Time Complexity:** O(n)  
**Space Complexity:** O(n)

**Advantages:**
- Can potentially detect expiration earlier
- More complex logic

**Disadvantages:**
- Still need second pass for incomplete RPCs
- More complex to understand

## Related Problems
- Meeting Rooms
- Car Pooling
- Employee Free Time
- Merge Intervals

