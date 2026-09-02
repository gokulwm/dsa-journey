# Longest SubArray with At Most K Distinct Characters - LeetCode 340

## The Problem

Given an array of integers `nums` and an integer `k`, find the length of the longest contiguous subarray that contains **at most `k` distinct elements**.

**Example:**
`nums = [1, 2, 1, 2, 3], k = 2`

- `[1, 2, 1, 2]` → 2 distinct elements (1, 2) → valid, length 4
- `[1, 2, 1, 2, 3]` → 3 distinct elements (1, 2, 3) → invalid, exceeds `k`

So the answer is `4`.

The key word is "at most" — the window is allowed to have fewer than `k` distinct elements too. It just can't have more.

## Step 1: Brute Force

Check every possible subarray, count its distinct elements, and keep the longest one that satisfies the constraint.

```java
static int longestBruteForce(int[] nums, int k) {
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        Set<Integer> seen = new HashSet<>();
        for (int j = i; j < nums.length; j++) {
            seen.add(nums[j]);
            if (seen.size() <= k) {
                max = Math.max(max, j - i + 1);
            } else {
                break;
            }
        }
    }
    return max;
}
```

**Complexity:** O(n²) time — for every starting index `i`, we extend `j` outward and rebuild the distinct-count picture from scratch. O(n) space for the `HashSet` in the worst case.

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes work because it forgets everything and restarts every time `i` moves. But notice: as the window's right edge grows, the distinct-count only changes in predictable, incremental ways — you're either introducing a new element (count goes up) or repeating one you've already seen (count stays the same).

That means you don't need to recompute distinctness from scratch on every step. You can maintain a running frequency map as the window expands one element at a time from the right. The moment the number of distinct keys exceeds `k`, the window has become invalid — and instead of restarting, you can shrink it from the left just enough to become valid again, decrementing frequencies as elements leave the window.

This converts the problem into the classic **variable-size sliding window**: expand right, and shrink left only when a constraint is violated. Since every element enters and leaves the window at most once, this brings the whole pass down to O(n).

The one subtlety: when shrinking, an element's frequency has to drop all the way to zero before it stops counting as "in the window" — removing one instance of a repeated element doesn't reduce the distinct count.

## My Optimized Solution

```java
static int longest(int[] nums, int k)
{
    HashMap<Integer, Integer> freq = new HashMap<>();
    int left = 0, count = 0;
    int max = 0;
    for(int right = 0; right < nums.length; right++)
    {
        if(!freq.containsKey(nums[right]))
            count++;
        freq.put(nums[right], freq.getOrDefault(nums[right], 0) + 1);
        while(count > k)
        {
            freq.put(nums[left], freq.get(nums[left]) - 1);
            if(freq.get(nums[left]) == 0)
                count--;
            left++;
        }
        max = Math.max(max, right - left + 1);
    }
    return max;
}
```

**Walking through the logic:**

1. `freq` is a frequency map — not just a membership set — because shrinking needs to know *when a count hits zero*, not just whether the element was ever seen.
2. `count` is a separate running tally of **distinct** elements currently in the window. It's cheaper to maintain incrementally than to call `freq.size()` on every iteration.
3. For each new `nums[right]`: if it's not already a key in `freq`, it's a new distinct element, so `count++`. Then its frequency is updated (or inserted) unconditionally.
4. If `count > k`, the window is invalid. The `while` loop shrinks from `left`: decrement the frequency of `nums[left]`, and only decrement `count` if that frequency reaches exactly `0` — meaning that element has fully left the window, not just lost one occurrence.
5. `max` is updated **after** the while loop, so it only ever records window sizes that are already valid (`count <= k`). Updating it before the shrink would risk recording an invalid window.

## Comparison

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(n²) | O(n) |
| Optimized (Sliding Window) | O(n) | O(n) |

## A Small Thing Worth Noting

If `max` is initialized to `Integer.MIN_VALUE` here instead of `0` -> For this problem it doesn't matter — LeetCode guarantees `nums.length >= 1`, so the loop always runs at least once and `max` gets a real value. But it's worth building the habit of defaulting to `int max = 0;` in sliding window problems generally: window lengths are never negative, and `0` is the honest answer for an empty input, whereas `MIN_VALUE` would silently leak through as a nonsense result if the array ever were empty.

The other detail worth carrying forward: this is the same "frequency map + separate distinct counter" split that shows up whenever a window's validity depends on *how many kinds* of things it contains rather than *how many* of one specific thing. Contrast this with Max Consecutive Ones III, where a single integer counter was enough because the constraint was one-dimensional (count of zeros). Here the constraint is about the count of distinct keys, so the frequency map is doing real work — it's not overkill the way it would be in a problem like LC 3, where a `HashSet` alone is sufficient.
