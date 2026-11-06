# Count Zero Request Servers - LeetCode 2747

## The Problem

You have `n` servers numbered `1..n` and a log of requests, where each entry `[server_id, time]` says that server received a request at that time. You're also given an integer `x` and an array of `queries`. For each `queries[i]`, count the servers that received **no requests at all** during the inclusive time interval `[queries[i] - x, queries[i]]`.

**Example:** `n = 3`, `logs = [[1,3],[2,6],[1,5]]`, `x = 5`, `queries = [10, 11]`

- Query `10` looks at `[5, 10]`: servers 1 (time 5) and 2 (time 6) are busy, server 3 is idle ✅
- Query `11` looks at `[6, 11]`: only server 2 (time 6) is busy, servers 1 and 3 are idle ✅

Output: `[1, 2]`

The tricky part is that every query defines its own window, the windows overlap heavily, and the queries arrive in no particular order, so answering each one from scratch is wasteful.

---

## Step 1: Brute Force

Take each query on its own: scan the entire log, mark every server that has a request inside the window, and subtract the number of marked servers from `n`.

```java
class Solution {
    public int[] countServers(int n, int[][] logs, int x, int[] queries) {
        int[] res = new int[queries.length];
        for (int i = 0; i < queries.length; i++) {
            int start = queries[i] - x, end = queries[i];
            boolean[] active = new boolean[n + 1];
            int busy = 0;
            for (int[] log : logs) {
                if (log[1] >= start && log[1] <= end && !active[log[0]]) {
                    active[log[0]] = true;
                    busy++;
                }
            }
            res[i] = n - busy;
        }
        return res;
    }
}
```

**Time: O(q · (m + n))**, where `m` is the number of logs and `q` the number of queries. Each query scans all `m` logs and allocates a fresh `boolean[n + 1]`.
**Space: O(n)**. One marker array, reused only in the sense that it is rebuilt per query.

With `n`, `m` and `q` all allowed to reach 10⁵, that is on the order of 10¹⁰ operations, far too slow.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force throws away everything it learned from the previous query. But look at what the queries actually are: every window has the same width `x`, so they are all the **same window slid to different positions on the time axis**. Two neighbouring queries, say 10 and 11, share almost their entire window; only a sliver enters on the right and a sliver leaves on the left.

That suggests processing the queries in **increasing order**. Then the window only ever moves rightwards, so each log enters the window exactly once and leaves it exactly once. If the logs are sorted by time, this becomes a classic two-pointer sweep: a `right` pointer that pulls in logs whose time is at most the query, and a `left` pointer that drops logs whose time has fallen below `query - x`. Total pointer movement across all queries is O(m), not O(q · m).

What do we maintain while the window slides? The answer is "servers with zero requests in the window", so we need to know, for each server, whether it still has any request inside. A plain boolean is not enough, because a server can have **several** requests in the window, and when one of them expires the server is still busy if another remains. So we keep a **count of in-window requests per server**. A server turns busy when its count goes from 0 to 1 and turns idle again when it goes from 1 to 0. A single running number of idle servers, adjusted only at those two transitions, then lets every query read its answer in O(1).

Three small details fall out of this. First, sorting the queries destroys their original order, so we must remember where each query came from in order to write its answer back to the right slot. Second, the interval is inclusive on both ends, so a log enters when `time <= query` and leaves only when `time < query - x`. Third, the `left` pointer must never overtake `right`, since we can only remove a log that was actually added.

---

## My Optimized Solution

```java
class Solution {
    public int[] countServers(int n, int[][] logs, int x, int[] queries) {
        Arrays.sort(logs, (a, b)->Integer.compare(a[1], b[1]));
        HashMap<Integer, Integer> resMap = new HashMap<>();
        for(int i = 0;i < queries.length;i++)
            resMap.put(queries[i], i);
        Arrays.sort(queries);
        int left = 0, right = 0;
        HashMap<Integer, Integer> map = new HashMap<>();
        int count = n;
        int[] res = new int[queries.length];
        int i = 0;
        for(;i < queries.length;i++)
        {
            while(right < logs.length && logs[right][1] <= queries[i])
            {
                if(!map.containsKey(logs[right][0]))
                    count--;
                map.put(logs[right][0], map.getOrDefault(logs[right][0], 0) + 1);
                right++;
            }
            while(left < right && logs[left][1] < queries[i] - x)
            {
                if(map.get(logs[left][0]) == 1)
                {
                    count++;
                    map.remove(logs[left][0]);
                }
                else
                    map.put(logs[left][0], map.get(logs[left][0]) - 1);
                left++;
            }
            res[resMap.get(queries[i])] = count;
        }
        return res;
    }
}
```

### Walking through the logic

1. **`Arrays.sort(logs, ...)`** orders the logs by timestamp (`a[1]`), which is what makes the two-pointer sweep possible.
2. **`resMap`** records, for each query value, its original index, **before** `queries` gets sorted. This is how each answer finds its way back to the right position in the output.
3. **`map`** holds, for each server currently inside the window, how many of its requests are in the window. **`count`** is the number of idle servers and starts at **`n`**, since an empty window has no busy servers.
4. **`while(right < logs.length && logs[right][1] <= queries[i])`** pulls every log with time up to the query into the window. If the server isn't in `map` yet (**`!map.containsKey(...)`**), it just went from idle to busy, so **`count--`**. Then its request count is incremented with **`getOrDefault(...) + 1`**.
5. **`while(left < right && logs[left][1] < queries[i] - x)`** evicts logs older than the window start. The **`left < right`** guard ensures we never evict a log that was never added.
6. **`if(map.get(logs[left][0]) == 1)`** means this was the server's last request in the window, so it becomes idle again: **`count++`** and remove it from `map`. Otherwise the server still has other requests inside, so we only decrement its count.
7. **`res[resMap.get(queries[i])] = count`** stores the current idle-server count at the query's original index.

**Trace on `n = 3`, `logs = [[1,3],[2,6],[1,5]]`, `x = 5`, `queries = [10, 11]`:**

Sorted logs: `[1,3], [1,5], [2,6]`. Initially `count = 3`, `map = {}`.

| query | window | logs added (right) | logs removed (left) | map after | count | res |
|-------|--------|--------------------|---------------------|-----------|-------|-----|
| 10 | [5, 10] | `[1,3]`, `[1,5]`, `[2,6]` | `[1,3]` (3 < 5) | {1:1, 2:1} | 1 | res[0] = 1 |
| 11 | [6, 11] | none | `[1,5]` (5 < 6) | {2:1} | 2 | res[1] = 2 |

Result: `[1, 2]` ✅

> **Note (no change made to the code):**
> - `Arrays` and `HashMap` need `import java.util.*;`, which LeetCode adds automatically but a local file would not.
> - **Possible bug with duplicate queries.** `resMap.put(queries[i], i)` overwrites earlier entries, so if the same value appears twice (for example `queries = [10, 10]`), both iterations write to the *last* index and the earlier index stays at its default `0`. The problem does not promise distinct queries, so this is worth testing. A common fix is to sort an array of indices by query value instead of using a value-to-index map.
> - `queries` is sorted in place, so the caller's input array is mutated. Harmless on LeetCode, but worth knowing.
> - `map` could be a plain `int[n + 1]`, which avoids hashing and boxing for a constant-factor speedup. It would also let `count` be derived rather than tracked, though the asymptotic complexity would not change.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (rescan all logs per query) | O(q · (m + n)) | O(n) | Forgets everything between queries |
| **Sort + Two Pointers + HashMap (mine)** | **O((m + q) log(m + q))** | **O(n + q)** | Sorting dominates; the sweep itself is O(m + q) |

Here `m` is the number of logs and `q` the number of queries. The sweep is linear, so the cost is the two sorts, and there is no asymptotically better tier to add.

---

## A Small Thing Worth Noting

The key move here is not the sliding window itself, it's **answering the queries offline**. Queries are independent of each other, so we are free to reorder them, and sorting them turns a pile of unrelated windows into one window that only moves forward. The price is bookkeeping: you have to remember each query's original position to write the answer back correctly (and, as the note above shows, that bookkeeping is exactly where duplicates can bite).

The recognition rule: **"many independent range or threshold queries over sorted data" → sort the queries too and sweep once**, instead of re-answering each from scratch. Note also that this window slides over *time values*, not array indices, so the two pointers walk the sorted logs rather than a fixed-width index range. Related problems that use the same offline idea are LC 1851 (Minimum Interval to Include Each Query) and LC 2070 (Most Beautiful Item for Each Query), and the "per-key count in a window, track when it hits zero" pattern is the same one behind LC 3 and LC 340 in this repo.
