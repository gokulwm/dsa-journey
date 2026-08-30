# Minimum Size Subarray Sum - LeetCode 209

## The Problem

Given an array of positive integers `nums` and a positive integer `target`, find the length of the shortest contiguous subarray whose sum is greater than or equal to `target`. If no such subarray exists, return `0`.

**Example:** `target = 7`, `nums = [2, 3, 1, 2, 4, 3]`

Walking through a few candidate windows by hand:

- `[2, 3, 1, 2]` → sum = 8 ≥ 7 ✅, length 4
- `[3, 1, 2, 4]` → sum = 10 ≥ 7 ✅, length 4
- `[1, 2, 4]` → sum = 7 ≥ 7 ✅, length 3
- `[2, 4, 3]` → sum = 9 ≥ 7 ✅, length 3
- `[4, 3]` → sum = 7 ≥ 7 ✅, length 2 ← shortest

Answer: **2**, from the subarray `[4, 3]`.

Unlike fixed-size sliding window problems, the window size itself is what we're trying to minimize — it grows and shrinks as we scan, rather than staying constant.

## Step 1: Brute Force

The most direct approach is to check every possible starting point, and for each one, keep extending the subarray to the right until the sum reaches `target`, recording the length whenever it does.

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;
        int minLen = Integer.MAX_VALUE;

        for (int start = 0; start < n; start++) {
            int sum = 0;
            for (int end = start; end < n; end++) {
                sum += nums[end];
                if (sum >= target) {
                    minLen = Math.min(minLen, end - start + 1);
                    break;
                }
            }
        }

        return minLen == Integer.MAX_VALUE ? 0 : minLen;
    }
}
```

**Complexity:**
- Time: `O(n^2)` — for every starting index, we potentially scan the rest of the array.
- Space: `O(1)`

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes work because every time we move `start` one step forward, we throw away everything we knew about the previous window and recompute its sum from scratch. But notice something: once we've expanded `end` far enough for the sum to cross `target`, we don't need to restart the sum from zero when `start` moves — we can just subtract `nums[start]` from the running total.

This observation is the seed of the sliding window technique. Since all values in `nums` are positive, the sum is *monotonic*: growing the window (moving `right`) only ever increases the sum, and shrinking it (moving `left`) only ever decreases it. That monotonicity is what makes the two-pointer approach valid here — there's no risk of a negative number undoing progress, so once a window's sum is large enough, we can greedily shrink it from the left to see how much smaller it can get before dropping below `target`.

So instead of restarting for every `start`, we maintain one running window with two pointers, `left` and `right`. We grow `right` to bring the sum up, and once the sum is at least `target`, we shrink from `left` as much as possible while still satisfying the condition — recording the window length each time. This turns the nested-restart pattern into a single pass where each pointer only ever moves forward, giving linear time overall.

## My Optimized Solution

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int sum = 0;
        int right = 0;
        int left = 0;
        while(sum < target && right < nums.length)
            sum += nums[right++];
        if(sum < target) return 0;
        int len = right - left;
        while(right < nums.length)
        {
            while(sum >= target)
            {
                len = Math.min(len, right - left);
                sum -= nums[left++];
            }
            sum += nums[right++];
        }
        while(sum >= target)
            {
                len = Math.min(len, right - left);
                sum -= nums[left++];
            }
        return len;
    }
}
```

**Walking through the logic:**

1. **Build the first valid window.** The initial `while(sum < target && right < nums.length)` loop grows `right` and accumulates `sum` until either the sum reaches `target` or the array runs out. This is a one-time setup step, separate from the main sliding loop below.
2. **Bail out early if it's impossible.** If we ran out of array before the sum ever reached `target`, no valid subarray exists at all, so we return `0` immediately.
3. **Seed `len` with the first valid window's size.** `len = right - left` gives us something to compare against as we start shrinking.
4. **Main loop: alternately shrink and grow.** For each remaining position of `right`, the inner `while(sum >= target)` loop greedily shrinks the window from the left — recording `len` and subtracting `nums[left]` — for as long as the window stays valid. As soon as shrinking would drop the sum below `target`, that inner loop stops, and we grow the window by one on the right (`sum += nums[right++]`) to try to become valid again.
5. **Drain the window after the main loop ends.** Once `right` reaches the end of the array, there may still be one more round of shrinking possible on the current window, so the final `while(sum >= target)` block repeats the same shrink-and-record logic one last time.
6. **Return the smallest length found**, which by this point reflects the shortest valid window seen across every shrink step.

Every element is added to `sum` exactly once (as `right` advances) and subtracted exactly once (as `left` advances), so despite the nested-looking `while` loops, each pointer only ever moves forward across the whole array — giving a true single-pass sliding window.

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(n^2) | O(1) |
| Optimized (Sliding Window) | O(n) | O(1) |

## A Small Thing Worth Noting

This problem is a good example of why the *direction* of monotonicity matters for sliding window validity, not just the presence of a two-pointer pattern. The greedy shrink-while-valid step only works because every `nums[i]` is positive — shrinking the window can only ever decrease the sum, never increase it, so there's no scenario where giving up an element on the left accidentally helps us. If negative numbers were allowed, this exact technique would break, because shrinking the window could sometimes *increase* the sum (by removing a negative value), making the greedy "shrink until invalid" logic unsound — that variant instead needs a different tool entirely (like a prefix-sum + monotonic deque). It's worth remembering that "sliding window works here" is really a claim about the structure of the data, not just the shape of the problem.
