# Number of Sub-arrays of Size K and Average ≥ Threshold - LeetCode 1343

## The Problem

We are given an array `arr`, a window size `k`, and a threshold value `t`. We need to count how many subarrays of length `k` have an average that is **greater than or equal to** `t`.

**Example:**
```
arr = [2, 2, 2, 2, 5, 5, 5, 8], k = 3, t = 4
```
Check every group of 3 consecutive numbers:
- `[2, 2, 2]` → avg = 2 → below threshold
- `[2, 2, 2]` → avg = 2 → below threshold
- `[2, 2, 5]` → avg = 3 → below threshold
- `[2, 5, 5]` → avg = 4 → meets threshold ✅
- `[5, 5, 5]` → avg = 5 → meets threshold ✅
- `[5, 5, 8]` → avg = 6 → meets threshold ✅

Answer: `3` subarrays meet the threshold.

## Step 1: Brute Force

Same first instinct as before — check every window of size `k` directly, sum it up, find the average, and see if it clears the threshold.

```java
class Solution {
    public int numOfSubarrays(int[] arr, int k, int t) {
        int count = 0;
        for (int i = 0; i <= arr.length - k; i++) {
            int sum = 0;
            for (int j = i; j < i + k; j++)
                sum += arr[j];
            int avg = sum / k;
            if (avg >= t)
                count++;
        }
        return count;
    }
}
```

**How it works:** the outer loop `i` fixes the starting index of the window, and the inner loop `j` sums up all `k` elements of that window. Once I have the sum, I get the average and check it against `t`. If it passes, I count it.

**Complexity:**
- Time: `O(n * k)` — for each of the `n - k + 1` windows, the inner loop does `k` work to sum it.
- Space: `O(1)`

Just like the last problem, this recomputes the sum of overlapping elements again and again for every window. Most of the work is repeated — only one element actually changes between consecutive windows.

## Step 2: Thinking Toward an Optimized Approach

Same idea as Maximum Average Subarray I — `k` is fixed, so the window only moves, it never resizes. That means I don't need to re-sum the whole window every time. I can carry the sum forward and just adjust it:
```
new_sum = old_sum - (element leaving from the left) + (element entering from the right)
```

This turns the inner `O(k)` work into `O(1)` per step, so the whole thing becomes a single pass sliding window.

## My Optimized Solution

```java
class Solution {
    public int numOfSubarrays(int[] arr, int k, int t) {
        int l = 0, r = k, sum = 0, curr, count = 0;

        // Step 1: sum of the very first window
        for (int i = 0; i < k; i++)
            sum += arr[i];
        curr = sum / k;
        if (curr >= t)
            count += 1;

        // Step 2: slide the window across the rest of the array
        while (r < arr.length) {
            sum = sum - arr[l] + arr[r];
            curr = sum / k;
            if (curr >= t)
                count += 1;
            l++;
            r++;
        }

        return count;
    }
}
```

**Walking through the logic:**
1. **Set up the first window.** Sum the first `k` elements, get its average (`curr`), and check it against `t` right away — this is window #1, so it needs to be checked before the loop starts.
2. **Two pointers, `l` and `r`.** `l` is the element about to leave the window, `r` is the element about to enter it. `r` starts at `k` since the first window already covers indices `0` to `k-1`.
3. **Slide one step at a time.** Update `sum` by removing `arr[l]` and adding `arr[r]`, then recompute the average and compare it to `t`. If it qualifies, bump `count`.
4. **Move both pointers forward** after checking, so the window shifts by exactly one position for the next iteration.
5. **Return `count`** — the total number of windows that met the threshold.

**Complexity:**
- Time: `O(n)` — each element enters and leaves the running sum exactly once.
- Space: `O(1)` — just a running sum, two pointers, and a counter.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O(n * k) | O(n) |
| Space | O(1) | O(1) |
| Idea | Recompute sum for every window | Reuse previous window's sum, just adjust it |

## A Small Thing Worth Noting

The average here is computed using **integer division** (`sum / k`), not `double`. At first this looks like it could cause rounding errors — but it actually works out fine. Since `t` is also an integer, checking `sum / k >= t` using integer (floor) division is mathematically the same as checking `sum >= t * k`. So no precision is lost here, unlike the previous problem where the answer itself needed to be a `double`.
