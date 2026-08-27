# Variable-Size Sliding Window (Two Pointers — Same Direction)

## Idea

Both pointers still move in the **same direction**, but unlike the fixed window, the window size is **not predetermined** — it expands and shrinks dynamically based on whether the current window satisfies a condition.

Instead of a window that always stays at length `k`, here `right` keeps pushing forward to grow the window, and `left` only moves when the window needs to be fixed. The size of `[left..right]` at any point is the answer to "how big/small can a valid window be here?" — not a fixed number decided in advance.

```
abcabcbb
l r                 -> "a" (valid, len 1)
l    r              -> "abc" (valid, len 3)
   l  r             -> duplicate 'a' -> shrink l
     l r            -> "bc" (valid again)
```

This replaces brute-force enumeration of every possible subarray/substring (`O(n^2)` or `O(n^3)`) with a single pass where each element is added once and removed at most once, giving `O(n)`.

## When to use it

- The problem talks about a **contiguous subarray or substring** (not subsequence, not subset — order and consecutiveness both matter).
- The window size is **not given directly** — instead the problem asks for:
  - "**longest** / **maximum**" window satisfying a condition, or
  - "**shortest** / **minimum**" window satisfying a condition
- Keywords like *at most K distinct*, *no duplicates*, *at most K zeros/violations*, *sum ≥ target*, *contains all characters of t* are strong signals.

This only works for **subarrays/substrings**, not subsequences or subsets — a subsequence doesn't require elements to be next to each other, so "shrinking from the left" doesn't make sense for it.

## Core Invariant

- Maintain `left` and `right`, with `right` always moving forward across the array/string.
- At each step, first **include** `arr[right]` into whatever bookkeeping structure you're using (count, sum, frequency map, set).
- Then check window validity, and either:
  - shrink from `left` because the window became **invalid**, or
  - shrink from `left` **while** it's still **valid**, to squeeze out a smaller answer.
- Which of these two you do depends entirely on whether you want the **longest** or the **shortest** valid window — this is the one decision that changes everything else about the code.

## Two Variants — This Is the Most Important Distinction

| | Longest / Maximum Window | Shortest / Minimum Window |
|---|---|---|
| Goal | Maximize window size | Minimize window size |
| Expansion | Expand freely with `right` | Expand until the window becomes valid |
| Shrinking | Shrink **only when invalid** | Shrink **aggressively while still valid** |
| When to update answer | After fixing an invalid window | While the window is valid, before it breaks |
| Mental model | Rubber band — stretch as far as possible, pull back only when it snaps | Tighten a knot — as soon as it's valid, keep pulling it tighter until it's about to fail |

**Memory trick:**
- Longest problems → *don't shrink unless forced.*
- Shortest problems → *shrink whenever you can.*

### Variant 1 — Longest / Maximum Window

**Template**

```java
int left = 0, maxLen = 0;

for (int right = 0; right < n; right++) {
    add(arr[right]);                 // expand: include current element

    while (windowInvalid()) {        // only shrink when broken
        remove(arr[left]);
        left++;
    }

    maxLen = Math.max(maxLen, right - left + 1); // update AFTER fixing
}
```

### Variant 2 — Shortest / Minimum Window

**Template**

```java
int left = 0, minLen = Integer.MAX_VALUE;

for (int right = 0; right < n; right++) {
    add(arr[right]);                  // expand until valid

    while (windowValid()) {           // shrink aggressively while still valid
        minLen = Math.min(minLen, right - left + 1); // update BEFORE breaking
        remove(arr[left]);
        left++;
    }
}
```

## Common Data Structures Used

| Structure | Used when |
|---|---|
| `HashSet` | Checking for duplicates / uniqueness (e.g. no repeating characters) |
| `HashMap` (frequency map) | Tracking counts of multiple distinct elements (e.g. at most K distinct, or matching a target frequency like in Minimum Window Substring) |
| Plain counter variable | Simple numeric conditions (e.g. sum ≥ target, count of zeros ≤ K) |
| Fixed-size array (size 26/128) | Character-frequency problems over a known alphabet — faster than a HashMap |

**A useful rule of thumb:** if you're just checking "has this appeared before," a `Set` is enough. If you need to know "which elements are still missing" or "how many of X do I currently have," you need a frequency `Map` — a `Set` alone can't answer that.

## Complexity

| Aspect | Cost |
|---|---|
| Time | `O(n)` — `right` moves forward `n` times; `left` moves forward at most `n` times total across the entire run (it never resets backward), so total pointer movement is `≈ 2n` |
| Space | `O(1)` for simple counters, `O(k)` or `O(alphabet size)` for a frequency map/set |

## Why This Beats Brute Force

Brute force checks every possible window explicitly — for every starting point, extend forward and re-validate from scratch, which is `O(n^2)` at best (and `O(n^3)` if re-validating a window itself takes `O(n)`). The sliding window avoids this by **never re-checking a window from scratch**: it reuses the previous window's state and only accounts for the single element entering and the single element(s) leaving. This is the same "add/remove, don't rebuild" principle from the fixed-size window — just applied with a dynamic boundary instead of a fixed one.

## Pitfalls

- **Mixing up the shrink condition.** Using `while (windowInvalid())` for a minimum-window problem (or vice versa) silently produces the wrong answer instead of crashing — always double check whether the loop condition should be "while broken" or "while still valid."
- **Updating the answer at the wrong point.** For longest, update *after* the shrink loop. For shortest, update *inside* the shrink loop, before `left` advances.
- **Off-by-one on length.** Always `right - left + 1`, not `right - left`.
- **Forgetting this only works for contiguous data.** Subsequences and subsets don't have a "left boundary" to shrink — this pattern doesn't apply to them.
- **Not handling the case where no valid window exists at all** (e.g. `t` has a character not present anywhere in `s`) — decide what the function should return (`0`, `-1`, `""`) before writing the loop.

## Quick Recognition Checklist

1. Is this about a **contiguous subarray/substring**? If not (subsequence/subset), this pattern doesn't apply.
2. Is the window size **not fixed**, and instead depends on a condition being true/false? → Variable sliding window.
3. Does the question ask for **longest/maximum**? → Expand freely, shrink only when invalid.
4. Does the question ask for **shortest/minimum**? → Expand until valid, then shrink aggressively while still valid.
