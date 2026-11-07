# Process Tasks Using Servers - LeetCode 1882

## The Problem

You have `n` servers, each with a **weight** (`servers[i]`), and `m` tasks, each with a **duration** (`tasks[j]`). Task `j` enters a queue at second `j`. Tasks are handled strictly in order. When a task is ready to be assigned, it goes to the free server with the **smallest weight**, and among equal weights the **smallest index**. A server that takes a task at second `t` is busy until second `t + tasks[j]`.

If no server is free when a task is at the front of the queue, it waits until some server frees up. If several servers free up at the same second, the waiting tasks are assigned in order, each one following the weight-then-index rule. Return, for each task, the index of the server that ran it.

**Example:** `servers = [3, 1]`, `tasks = [4, 4, 1, 1]`

- Second 0: task 0 → server 1 (weight 1), busy until 4.
- Second 1: task 1 → server 0 (only free one), busy until 5.
- Task 2 arrives at 2 but nobody is free. At second 4 server 1 frees up → task 2 runs on server 1, busy until 5.
- Task 3 arrives at 3, still waits. At second 5 both servers are free → server 1 wins on weight.

Output: `[1, 0, 1, 1]`

The tricky part is that "current time" is not simply the task index. The clock can jump forward past `i` whenever every server is busy, and it must never go backward afterwards.

---

## Step 1: Brute Force

Simulate the statement literally. Keep, for each server, the second at which it becomes free. For every task, work out the current time, scan all servers for the best free one, and if none exists, jump to the earliest finish time and scan again.

```java
class Solution {
    public int[] assignTasks(int[] servers, int[] tasks) {
        int n = servers.length, m = tasks.length;
        long[] freeAt = new long[n];
        int[] ans = new int[m];
        long time = 0;
        for (int j = 0; j < m; j++) {
            time = Math.max(time, j);
            int best = pick(servers, freeAt, time);
            if (best == -1) {
                long earliest = Long.MAX_VALUE;
                for (int s = 0; s < n; s++)
                    earliest = Math.min(earliest, freeAt[s]);
                time = earliest;
                best = pick(servers, freeAt, time);
            }
            freeAt[best] = time + tasks[j];
            ans[j] = best;
        }
        return ans;
    }

    // best free server at 'time': smallest weight, then smallest index
    private int pick(int[] servers, long[] freeAt, long time) {
        int best = -1;
        for (int s = 0; s < servers.length; s++)
            if (freeAt[s] <= time && (best == -1 || servers[s] < servers[best]))
                best = s;
        return best;
    }
}
```

The strict `<` combined with scanning indices in increasing order gives the index tie-break for free.

**Time: O(m · n)**. Every task does a full scan of all `n` servers (a second scan in the waiting case is still O(n)). With `n, m` up to 2·10⁵ that is around 4·10¹⁰ steps, far too slow.
**Space: O(n)**. The `freeAt` array (plus the output).

---

## Step 2: Thinking Toward an Optimized Approach

The brute force spends all its time answering two questions by scanning: "which servers are free right now?" and "which of them is the best?" Both are really *ordering* questions, and a priority queue answers an ordering question in logarithmic time, so the natural move is to split the servers into two groups and keep each group in its own heap.

The **free** group only ever needs to answer "who is best?", and best is defined by weight, then index. So its heap is ordered by `(weight, index)`. The **busy** group only ever needs to answer "who finishes next?", so its heap is ordered by finish time. The busy heap also has to store the weight and index, because that is the information needed to rebuild a server entry when it moves back to the free pool, so each entry is `(finish time, weight, index)`. Ties on finish time are then broken by weight and index, which keeps the release order deterministic.

With the two heaps, the simulation becomes a loop over tasks with a clock. Before assigning task `i`, move every busy server whose finish time has arrived back into the free heap. If the free heap is now non-empty, pop the best server. If it is still empty, nothing can happen until the earliest busy server finishes, so the clock should **jump** straight to that finish time (no need to tick second by second), release everything finishing at that second, and then pop.

The detail that decides correctness is how the clock itself advances. The clock must be at least `i`, because task `i` cannot start before it arrives, but it may already be ahead of `i` from an earlier jump, and it must not be dragged backward or pushed forward artificially. That is a "take the larger of the two" operation, not an increment. Each task costs a constant number of heap operations, so the whole thing is `O((n + m) log n)`.

---

## My Optimized Solution

```java
class Solution {
    static class Pair
    {
        int value;
        int index;
        public Pair(int v, int i)
        {
            value = v;
            index = i;
        }
    }
    public int[] assignTasks(int[] servers, int[] tasks) 
    {
        PriorityQueue<Pair> free = new PriorityQueue<>((a, b) -> {
            if(a.value != b.value)
                return Integer.compare(a.value, b.value);
            return Integer.compare(a.index, b.index);
        });
        for(int i = 0;i < servers.length;i++)
            free.offer(new Pair(servers[i], i));
        int[] ans = new int[tasks.length];
        PriorityQueue<long[]> assigned = new PriorityQueue<>((a, b) -> {
            if(a[0] != b[0]) //time comparison - here time is the final time of completion
                return Long.compare(a[0], b[0]);
            else if(a[1] != b[1]) //value comparison
                return Long.compare(a[1], b[1]);
            return Long.compare(a[2], b[2]); //index comparison
        });
        long time = 0;
        long arr[];
        Pair p;
        for(int i = 0;i < tasks.length;i++)
        {
            time = Math.max(time, i);
            while(!assigned.isEmpty() && time == assigned.peek()[0])
            {
                arr = assigned.poll();
                free.offer(new Pair((int)arr[1], (int)arr[2]));
            }


            if(free.isEmpty())
            {
                time = assigned.peek()[0];
                while(!assigned.isEmpty() && time == assigned.peek()[0])
                {
                    arr = assigned.poll();
                    free.offer(new Pair((int)arr[1], (int)arr[2]));
                }
            }

            p = free.poll();
            assigned.offer(new long[]{time + tasks[i], p.value, p.index});
            ans[i] = p.index;
            //time++;
        }
        return ans;
    }
}
```

### Walking through the logic

1. **`free`** is a min-heap of `Pair(value, index)` ordered by weight, then index, so `free.poll()` is always exactly the server the problem wants. It starts with every server in it.
2. **`assigned`** is a min-heap of `long[]{finishTime, value, index}` ordered by finish time, then weight, then index. Its top is always the next server to become free. `long` is used because `time + tasks[i]` can reach roughly 4·10¹⁰, which overflows `int`.
3. **`time = Math.max(time, i)`** is the clock rule: task `i` cannot start before second `i`, but if an earlier wait already pushed the clock beyond `i`, it stays there.
4. **The first `while` loop** releases every busy server whose finish time equals the current `time`, converting each `long[]` back into a `Pair` and putting it into `free`.
5. **`if(free.isEmpty())`** handles the all-busy case. Since `free` is empty, every server is in `assigned`, so `assigned.peek()` is safe. **`time = assigned.peek()[0]`** jumps the clock to the earliest finish, and the inner `while` releases everyone finishing at exactly that second.
6. **`p = free.poll()`** takes the best server by weight then index. **`assigned.offer(new long[]{time + tasks[i], p.value, p.index})`** marks it busy until `time + tasks[i]`, and **`ans[i] = p.index`** records the answer.
7. **`//time++`** is deliberately commented out. The clock is not incremented per task; it is only ever set by `max(time, i)` and by the jump in step 5.

**Trace on `servers = [3, 1]`, `tasks = [4, 4, 1, 1]`:**

| i | after `max(time, i)` | free empty? | time after jump | server picked | pushed to `assigned` (finish, w, idx) |
|---|---|---|---|---|---|
| 0 | 0 | no | 0 | idx 1 (w=1) | (4, 1, 1) |
| 1 | 1 | no (idx 0 left) | 1 | idx 0 (w=3) | (5, 3, 0) |
| 2 | 2 | **yes** | **4** (release idx 1) | idx 1 | (5, 1, 1) |
| 3 | max(4, 3) = **4** | **yes** | **5** (release idx 1 and idx 0) | idx 1 (w=1 beats w=3) | (6, 1, 1) |

Result: `[1, 0, 1, 1]` ✅

Notice row 3: `i = 3` but the clock is already at 4, so `max` keeps it at 4 instead of resetting it to 3 or pushing it to 5.

> **Note (no change made to the code):** The release loops use `time == assigned.peek()[0]` rather than `<=`. This is correct here only because of an invariant: the clock never skips past a pending finish time, since it moves up by at most 1 through `max(time, i)` or jumps exactly to the smallest finish time. If the clock ever skipped over a pending finish time, that server would never be released. Using `<=` would make the code robust to that, but it is not needed for correctness as written. Also, `PriorityQueue` needs `import java.util.*;`, which LeetCode supplies automatically.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (scan all servers per task) | O(m · n) | O(n) | Literal simulation, about 4·10¹⁰ steps at max constraints |
| **Two Heaps: free + assigned (mine)** | **O((n + m) log n)** | **O(n)** | Free heap by (weight, index), busy heap by (finish, weight, index) |

---

## A Small Thing Worth Noting

This is an exact simulation of the process, and the part that needs care is how `time` is updated. It is tempting to write `time++` at the end of each iteration, as if one task is handled per second. That fails whenever there are no free servers: the clock jumps *beyond* `i` to the moment a server frees up, and the next task does not start at that moment plus one, it starts **at that very moment** (the same second can free several servers, and several waiting tasks can start then). So the clock must be initialized for the next task as the current `time`, not `time + 1`, and `Math.max(time, i)` is what lets it do that: it respects the arrival lower bound `i` while keeping any jump that already happened.

A concrete case: `servers = [1, 1]`, `tasks = [4, 3, 1, 1]`. Both servers free up at second 4. Task 2 takes server 0 from 4 to 5, and task 3 should take server 1 **also at second 4** (answer `[0, 1, 0, 1]`). An unconditional `time++` would move the clock to 5 before task 3, so task 3 would start at second 5 instead of 4. By then server 0 (finishing at 5) is free again too, the weight tie goes to the smaller index, and the answer becomes `[0, 1, 0, 0]` with wrong finish times.

The data-structure choice follows the same logic: the `free` heap is keyed on **(value, index)** because that is the selection rule, and the `assigned` heap is keyed on **(finish time, value, index)** because it must tell us who frees up next and hand servers back deterministically. A recognition rule: **"events happen at computed future times and you must always act on the earliest one" → two heaps (or a heap plus a clock), jumping time instead of ticking it.** The same pattern shows up in Single-Threaded CPU (LC 1834), Meeting Rooms III (LC 2402), and Meeting Rooms II (LC 253).
