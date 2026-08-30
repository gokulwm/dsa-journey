# Max Consecutive Ones III - LeetCode 1004

## The Problem

You're given a binary array `nums` and an integer `k`. You're allowed to flip at most `k` zeros to ones. You need to find the length of the longest contiguous subarray of all 1s you can get after doing those flips.

The key phrase is "at most `k`" — you don't have to use all your flips, but you can't exceed the budget.

**Example:**

```
nums = [1,1,1,0,0,0,1,1,1,1,0]
k = 2
```

If you flip the two 0s at indices 4 and 5, you get:

```
[1,1,1,0,1,1,1,1,1,1,0]
              ^-------^
```

The subarray from index 5 to index 9 (`[1,1,1,1,1,1]` — wait, let's be precise) — the longest stretch of 1s achievable is `[1,1,1,1,1,1]` spanning indices 5 to 10 after flipping index 9's neighbor zeros appropriately. The actual answer here is **6**, achieved by flipping the zeros at indices 4 and 5, giving a run from index 5 through index 10 (`nums[5..10]` after flips = six consecutive 1s).

The point isn't the exact indices — it's the idea: a valid window is any window where **the count of zeros inside it is ≤ k**. You're not physically flipping anything in code; you're just finding the longest window whose zero-count fits your budget.

## Step 1: Brute Force

The direct brute force is: for every possible starting point, extend as far right as possible while counting zeros, and stop the moment the zero count would exceed `k`.

```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int n = nums.length;
        int ans = 0;
        for (int start = 0; start < n; start++) {
            int zeroCount = 0;
            for (int end = start; end < n; end++) {
                if (nums[end] == 0) {
                    zeroCount++;
                }
                if (zeroCount > k) {
                    break;
                }
                ans = Math.max(ans, end - start + 1);
            }
        }
        return ans;
    }
}
```

**Complexity:**
- Time: O(n²) — for every start index, we walk forward until the budget breaks.
- Space: O(1) — just a running counter.

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes work because it re-scans overlapping regions from scratch for every new `start`. Notice that as `start` increases by one, the window we're allowed to consider only shrinks — it never has to re-examine elements further right than what a slightly-earlier start already reached. That's the signature of a problem where a two-pointer / sliding window can replace the nested loop.

The real insight is what makes a window "valid" here: a window `[left, right]` is valid exactly when the number of zeros inside it is ≤ k. This is a single, cheaply-maintained integer condition — you don't need to know *where* the zeros are, only *how many* there are in the current window. That's what allows you to track validity with one counter instead of re-scanning.

So the strategy becomes: expand `right` one step at a time, incrementing the zero counter whenever you include a zero. If the counter ever exceeds `k`, the window is no longer valid — shrink from `left` until it becomes valid again, decrementing the counter whenever the element leaving the window is a zero. At every point where the window is valid, its length is a candidate answer.

This only works cleanly because of a monotonic property: once you fix `right`, the smallest valid `left` for that `right` never needs to move backward as `right` increases. Each pointer only moves forward, over the whole array, giving O(n) total work instead of O(n²). This is the same "shrink-while-invalid" backbone as the general variable-size window template, just with the validity condition swapped from "no duplicate letters" or "at most K distinct" to "zero count within budget."

## My Optimized Solution

```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int count0 = 0;
        int left = 0;
        int ans = 0;
        for(int right = 0;right < nums.length;right++)
        {
            if(nums[right] == 0)
            count0 += 1;
            while(count0 > k)
            {
                if(nums[left] == 0)
                count0 -= 1;
                left += 1;
            }
            ans = Math.max(ans, right - left + 1);
        }
        return ans;
    }
}
```

**Walking through the logic:**

1. `count0` tracks how many zeros are currently inside the window `[left, right]`. `left` marks the window's start, `ans` tracks the best valid window length seen so far.
2. The `for` loop expands the window one element at a time by moving `right` forward. This is the only way the window ever grows.
3. If `nums[right]` is a zero, it counts against the budget, so `count0` is incremented.
4. The `while(count0 > k)` loop is the shrink step. As long as the window holds more zeros than allowed, `left` is pushed forward. Before advancing `left`, we check if the element being *removed* was a zero — if so, `count0` is decremented, since that zero is no longer inside the window.
5. Note this is a `while`, not an `if` — in this problem a single shrink step is always enough to restore validity (since each step removes at most one zero), but writing it as `while` keeps the pattern general and correct regardless of how the invalidity arose.
6. Once the `while` loop exits, `count0 <= k` is guaranteed, so `[left, right]` is a valid window. Its length `right - left + 1` is compared against `ans`.
7. This repeats until `right` has swept the whole array, and `ans` holds the longest valid window found.

## Comparison

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(n²) | O(1) |
| Optimized (Sliding Window) | O(n) | O(1) |

## A Small Thing Worth Noting

This problem is a good example of a window whose validity condition is a **count**, not a **set**. Contrast this with something like Longest Substring Without Repeating Characters, where you need a `Set`/`Map` because validity depends on *which* characters are present. Here, validity only depends on *how many* zeros are present — the positions and identities of the zeros don't matter at all once you've counted them. Whenever a problem's constraint reduces to "at most X occurrences of some countable property," you often don't need a hash structure at all; a single integer counter, incremented and decremented as elements enter and leave the window, is enough. It's worth checking, for every new sliding window problem, whether the validity condition is really a *count* in disguise before reaching for a `Map`.
