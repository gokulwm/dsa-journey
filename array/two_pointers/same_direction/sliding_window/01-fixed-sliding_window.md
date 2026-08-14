# Fixed-Size Sliding Window (Two Pointers — Same Direction)

## Idea

Both pointers move in the **same direction**, and the window between them always stays a **constant size `k`**. As the window slides forward by one step, exactly one element enters from the right and one element leaves from the left.

This replaces the brute-force approach of recomputing every window from scratch (`O(n * k)`), bringing it down to `O(n)` by *reusing* the work already done for the previous window.

```
[0  1  2  3  4  5  6]
 l________r              window size k = 3
    l________r
       l________r
          l________r
```

## When to use it

- The problem explicitly gives a **fixed window size `k`** (e.g. "subarray of size k", "window of length k").
- You need something computed **per window** — sum, average, max/min, a frequency count, or a validity check (like "does this window contain all distinct characters?").
- The window size **never changes** while scanning.

## Core Invariant

- `window = arr[l..r]`, and `r - l + 1 == k` at all times once the window is fully formed.
- Moving forward means: **add `arr[r]`**, then if the window has grown past `k`, **remove `arr[l]`** and increment `l`.
- Each element is added exactly once and removed exactly once → `O(n)` total work.

## General Template (Java)

```java
public int fixedSizeWindow(int[] arr, int k) {
    if (arr.length < k) return -1; // not enough elements

    int windowValue = 0; // could be sum, count, etc.
    int best = Integer.MIN_VALUE;

    // Step 1: build the first window
    for (int i = 0; i < k; i++) {
        windowValue += arr[i];   // "add" operation
    }
    best = windowValue;

    // Step 2: slide the window one step at a time
    for (int r = k; r < arr.length; r++) {
        windowValue += arr[r];          // element entering
        windowValue -= arr[r - k];      // element leaving
        best = Math.max(best, windowValue);
    }

    return best;
}
```

## Variant: Window Validity Using a Hash Structure

When the "value" you're tracking isn't a simple running sum but a **property of the whole window** (e.g. "all elements are distinct", "frequency matches a target"), keep a `HashMap`/`HashSet`/frequency array in sync as elements enter and leave.

```java
public boolean hasAllDistinct(int[] arr, int k) {
    Set<Integer> window = new HashSet<>();

    // first window
    for (int i = 0; i < k; i++) {
        if (!window.add(arr[i])) return false; // duplicate found
    }

    for (int r = k; r < arr.length; r++) {
        window.remove(arr[r - k]); // element leaving
        if (!window.add(arr[r])) return false; // element entering
    }

    return true;
}
```

The key trick here: don't rebuild the set from scratch for every window — only remove the outgoing element and add the incoming one.

## Complexity

| Aspect | Cost |
|---|---|
| Time | `O(n)` — each element enters and leaves the window exactly once |
| Space | `O(1)` for numeric aggregates, `O(k)` or `O(alphabet)` if using a frequency map/set |

## Pitfalls

- **Not handling `arr.length < k`** — always guard against this.
- **Recomputing the whole window every slide** instead of doing an incremental add/remove — this silently turns an `O(n)` solution into `O(n*k)`.
- **Off-by-one on the sliding boundary** — the element leaving the window is always at index `r - k`, not `r - k + 1` or `l`.
- **Integer overflow** on sums for large arrays/values — use `long` if needed.

## Quick Recognition Checklist

- Does the problem say "**window of size k**", "**subarray of exactly k elements**"? → Fixed sliding window.
- Do you need one value (sum/count/frequency) recomputed per window, reusing the previous window's work? → Fixed sliding window.
