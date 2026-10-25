# Subarray Sum Equals K - LeetCode 560

## The Problem

You're given an integer array `nums` and an integer `k`. Count how many **contiguous** subarrays have a sum exactly equal to `k`. The array can contain negative numbers and zeros, and subarrays that overlap are all counted separately.

**Example:** `nums = [1, 2, 3]`, `k = 3`

- `[1, 2]` sums to 3 ✅
- `[3]` sums to 3 ✅
- Everything else (`[1]`, `[2]`, `[2, 3]`, `[1, 2, 3]`) does not.

Output: `2`

The tricky part is not finding *a* subarray, it's counting *all* of them without checking every single one by hand.

---

## Step 1: Brute Force

The most direct translation of the problem: pick every start `i`, every end `j`, and add up the elements in between.

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            for (int j = i; j < nums.length; j++) {
                int sum = 0;
                for (int x = i; x <= j; x++) {
                    sum += nums[x];
                }
                if (sum == k) count++;
            }
        }
        return count;
    }
}
```

**Time: O(n³)**. There are O(n²) (start, end) pairs, and each one re-adds up to n elements.
**Space: O(1)**. Only a few integer variables.

The obvious first fix is that the inner `x` loop is pure waste. When `j` moves from `j` to `j + 1`, the new subarray sum is just the old sum plus `nums[j + 1]`. So we can keep a running sum per start index:

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            int sum = 0;
            for (int j = i; j < nums.length; j++) {
                sum += nums[j];
                if (sum == k) count++;
            }
        }
        return count;
    }
}
```

**Time: O(n²)**, **Space: O(1)**. This is the "Better" tier, but it still looks at every (start, end) pair individually.

---

## Step 2: Thinking Toward an Optimized Approach

Even the O(n²) version asks the same question over and over from scratch: "starting at `i`, does extending to `j` hit `k`?" For each start it walks forward and checks every end. That's a lot of pairs, and the pairs aren't independent, they overlap heavily.

Here's the way out. Instead of thinking about a subarray as "start at `i`, end at `j`," think about it as a **difference of two running totals**. If we define the prefix sum up to index `j` as the total of everything from the beginning through `j`, then the sum of the subarray from `i + 1` through `j` is simply `prefix[j] - prefix[i]`. A subarray ending at `j` sums to `k` exactly when some earlier prefix equals `prefix[j] - k`.

That flips the question. We no longer need to loop over all starting points for each end. We only need to know: **how many earlier prefix sums have the value `prefix[j] - k`?** That's a counting-by-value question, and a hash map answers it in O(1). So as we sweep left to right, we keep a map from "prefix sum value" to "how many times we've seen it," and at each position we look up `prefix - k`. Each lookup instantly accounts for every valid start for that end.

Two small details fall out of this reasoning. First, a subarray that starts at index 0 corresponds to subtracting a prefix of *nothing*, so we need to pretend an empty prefix with sum 0 has already been seen once. Second, we must look up `prefix - k` *before* inserting the current prefix, otherwise a subarray could be matched against itself when `k = 0`.

---

## My Optimized Solution

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        HashMap<Integer, Integer> presummap = new HashMap<>();
        int prefixsum = 0;
        presummap.put(0, 1);
        for(int i = 0;i < nums.length;i++)
        {
            prefixsum += nums[i];
            if(presummap.containsKey(prefixsum - k))
            {
                count += presummap.get(prefixsum - k);
            }
            presummap.put(prefixsum, presummap.getOrDefault(prefixsum, 0)+1);
        }
        return count;
    }
}
```

### Walking through the logic

1. **`presummap`** maps a prefix-sum value to the number of times that value has appeared so far. **`prefixsum`** is the running total as we sweep through the array.
2. **`presummap.put(0, 1)`** seeds the "empty prefix." This is what lets a subarray beginning at index 0 be counted: if the whole prefix up to `i` equals `k`, then `prefixsum - k = 0`, and that 0 must already be in the map.
3. **`prefixsum += nums[i]`** extends the running total to include the current element, so `prefixsum` is now the sum of `nums[0..i]`.
4. **`containsKey(prefixsum - k)`** asks: is there an earlier prefix whose removal leaves a subarray summing to `k`?
5. **`count += presummap.get(prefixsum - k)`** adds the *number of times* that earlier prefix occurred, because each occurrence is a different valid starting point. This is why the map stores counts, not just presence.
6. **`presummap.put(prefixsum, getOrDefault(...) + 1)`** records the current prefix for future positions. It happens *after* the lookup so the current prefix can't pair with itself.

**Trace on `nums = [1, 2, 3]`, `k = 3`:**

| i | nums[i] | prefixsum | look up (prefixsum − k) | found? | count | map after |
|---|---------|-----------|--------------------------|--------|-------|-----------|
| – | – | 0 | – | – | 0 | {0:1} |
| 0 | 1 | 1 | -2 | no | 0 | {0:1, 1:1} |
| 1 | 2 | 3 | 0 | yes (×1) → `[1,2]` | 1 | {0:1, 1:1, 3:1} |
| 2 | 3 | 6 | 3 | yes (×1) → `[3]` | 2 | {0:1, 1:1, 3:1, 6:1} |

Result: `2` ✅

> **Note (no change made to the code):** `HashMap` needs `import java.util.*;`, which LeetCode adds automatically but a local file would not. Also, `prefixsum` is an `int`; under this problem's constraints (up to 2·10⁴ elements, each at most 1000 in magnitude) it stays around 2·10⁷, so there's no overflow risk here, but it would be something to watch with larger inputs.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (re-sum every subarray) | O(n³) | O(1) | Recomputes overlapping sums |
| Better (running sum per start) | O(n²) | O(1) | Reuses the sum, still checks every pair |
| **Prefix Sum + HashMap (mine)** | **O(n)** | **O(n)** | One pass; trades memory for time |

---

## A Small Thing Worth Noting

This problem looks like a sliding window problem, but sliding window **can't** solve it. Sliding window relies on a monotonic property: expanding the window increases the sum, shrinking it decreases the sum, so you always know which way to move a pointer. That only holds when all elements are non-negative (or, as in LC 209, all positive). Here `nums` can contain negatives and zeros, so a larger window might have a *smaller* sum, and there's no safe rule for when to shrink.

Prefix sum + hashmap doesn't need that property at all. It only uses the algebraic identity `sum(i+1..j) = prefix[j] − prefix[i]`, which is true regardless of sign. A good recognition rule: **"count subarrays with sum equal to a target" + "negatives allowed" → prefix sum + hashmap, not sliding window.** The same idea extends to variations like subarrays divisible by `k` (store prefix sums mod `k`) and longest subarray with sum `k` (store the *first index* of each prefix instead of a count).
