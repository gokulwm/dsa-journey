# Trapping Rain Water - LeetCode 42

## The Problem

You're given an array where each number is the height of a bar of width 1. After it rains, water collects in the dips between taller bars. How many units of water are trapped in total?

The key question to ask is **"how much water sits on top of one single bar?"** Water above bar `i` can rise only as high as the *shorter* of the tallest bar to its left and the tallest bar to its right (anything higher spills over the shorter side). So:

```
water on bar i = min(maxLeft(i), maxRight(i)) - height[i]
```

**Example**

```
Input:  [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

| Index | 2 | 4 | 5 | 6 | 9 |
|---|---|---|---|---|---|
| Height | 0 | 1 | 0 | 1 | 1 |
| Water on top | 1 | 1 | 2 | 1 | 1 |

Every other bar holds 0, and 1 + 1 + 2 + 1 + 1 = **6**.

---

## Step 1: Brute Force

Apply the formula literally: for every bar, scan left to find the tallest bar up to it, scan right to find the tallest bar from it onward, and add up the difference.

```java
class Solution {
    public int trap(int[] height) {
        int n = height.length;
        int total = 0;
        for (int i = 0; i < n; i++) {
            int leftMax = 0, rightMax = 0;
            for (int j = 0; j <= i; j++)
                leftMax = Math.max(leftMax, height[j]);
            for (int j = i; j < n; j++)
                rightMax = Math.max(rightMax, height[j]);
            total += Math.min(leftMax, rightMax) - height[i];
        }
        return total;
    }
}
```

- **Time: O(n²)** - each of the `n` bars triggers two scans that together cost up to `n` steps.
- **Space: O(1)** - only a few integer variables.

---

## Step 2: Thinking Toward an Optimized Approach

Look at what the brute force actually recomputes. The value `maxLeft(i)` is the running maximum of a prefix, and `maxLeft(i+1)` differs from it by at most one comparison. Yet the brute force rebuilds that maximum from scratch for every bar. The same waste happens on the right side. Nothing about these maxima is specific to bar `i`; they are just prefix-maximum and suffix-maximum sequences being recomputed `n` times.

The first fix is to precompute both sequences once: one pass left to right for prefix maxima, one pass right to left for suffix maxima, and then a final pass that applies the formula. That drops the time to O(n), but it costs two extra arrays of size `n`. This is the natural "better" tier, and it pays for speed with memory.

The next question is whether those arrays are really needed. The formula takes the *minimum* of the two maxima, which means that for any bar, the taller side's exact value never matters. All that matters is that the taller side is at least as tall as the shorter side. So if we somehow know which side is the limiting one, we only need that side's running maximum, and we can finalize the bar's water without ever looking at the other side's exact maximum.

That suggests two pointers closing in from both ends, each carrying a running maximum for its own side. At any moment, compare the bars at the two pointers. The side with the shorter bar is the limiting side: everything between the pointers is guaranteed to have a boundary on the other side at least as tall as that bar. So we can safely settle the water for the bar next to the shorter pointer using only that pointer's running maximum, then advance that pointer. Each step permanently resolves one bar, so the whole thing is one pass with no arrays.

---

## My Optimized Solution

```java
class Solution {
    public int trap(int[] nums) {
     int leftArea = 0;
        int rightArea = 0;
        int left = 0, right = nums.length - 1, maxlHeight = nums[left], maxrHeight = nums[right];
        while(left < right)
        {
            if(nums[left] < nums[right])
            {
                if(left+1 < right)
                {
                    if(nums[left + 1] >= maxlHeight)
                        maxlHeight = nums[left+1];
                    else
                        leftArea += maxlHeight - nums[left+1];
                }
                left++;
            }
            else
            {
                if(left < right - 1)
                {
                    if(nums[right-1] >= maxrHeight)
                        maxrHeight = nums[right-1];
                    else
                        rightArea += maxrHeight - nums[right - 1];
                }
                right--;
            }
        }
        return leftArea + rightArea;   
    }
}
```

**Notes on the code (the code itself is left untouched):**

- The solution is correct. Nothing here is a bug.
- The parameter is named `nums` while the problem calls it `height`; purely cosmetic.
- `leftArea` and `rightArea` are never used separately, so a single `water` accumulator would do the same job.
- This is a **look-ahead** variant: the pointer stays on a "known" bar and the code settles the *neighbouring* bar (`left+1` or `right-1`). The more common formulation settles the bar at the pointer itself. Both are valid; yours just shifts which bar is considered "current".

### Walking through the logic

1. **Setup.** `left` and `right` start at the two ends. `maxlHeight` is the tallest bar seen from the left (initially `nums[0]`) and `maxrHeight` is the tallest seen from the right (initially the last bar). Invariant: `maxlHeight = max(nums[0..left])` and `maxrHeight = max(nums[right..n-1])`.
2. **Pick the limiting side.** If `nums[left] < nums[right]`, the left side is the shorter one, so the bar at `left+1` can be settled using only `maxlHeight`. Otherwise (including ties) the right side is limiting and the bar at `right-1` is settled using `maxrHeight`.
3. **Settle one bar on the left.** If `nums[left+1] >= maxlHeight`, that bar is a new wall, so it traps nothing and just raises `maxlHeight`. Otherwise it sits in a dip, and it holds `maxlHeight - nums[left+1]` units. Then `left++`.
4. **Settle one bar on the right.** This is the mirror image, using `maxrHeight` and the bar at `right-1`, followed by `right--`.
5. **The `left+1 < right` and `left < right - 1` guards.** When the pointers are adjacent, the "neighbour" is the other pointer's own bar, which has already been folded into that side's maximum. The guard stops that bar from being processed twice.
6. **Termination.** Each iteration moves one pointer inward, and every interior bar is settled exactly once. The loop ends when `left == right`, and the answer is `leftArea + rightArea`.

**Trace on `[0,1,0,2,1,0,1,3,2,1,2,1]`:**

| Step | left | right | Branch | Bar settled | Effect | Total |
|---|---|---|---|---|---|---|
| 1 | 0 | 11 | left | idx 1 (h=1) | new wall, `maxl=1` | 0 |
| 2 | 1 | 11 | right (tie) | idx 10 (h=2) | new wall, `maxr=2` | 0 |
| 3 | 1 | 10 | left | idx 2 (h=0) | +1 | 1 |
| 4 | 2 | 10 | left | idx 3 (h=2) | new wall, `maxl=2` | 1 |
| 5 | 3 | 10 | right | idx 9 (h=1) | +1 | 2 |
| 6 | 3 | 9 | right | idx 8 (h=2) | new wall | 2 |
| 7 | 3 | 8 | right | idx 7 (h=3) | new wall, `maxr=3` | 2 |
| 8 | 3 | 7 | left | idx 4 (h=1) | +1 | 3 |
| 9 | 4 | 7 | left | idx 5 (h=0) | +2 | 5 |
| 10 | 5 | 7 | left | idx 6 (h=1) | +1 | 6 |
| 11 | 6 | 7 | left | guard blocks (adjacent) | none | 6 |

Final answer: **6**.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Rescans both sides for every bar |
| Better (prefix/suffix max arrays) | O(n) | O(n) | Precomputes both maxima, trading memory for speed |
| Optimized (two pointers) | O(n) | O(1) | Carries only a running max per side and settles one bar per step |

---

## A Small Thing Worth Noting

The reason the two-pointer version works is a *one-sided certainty* argument: to decide the water on a bar you never need the exact value of the taller side's maximum, only a guarantee that it is at least as tall as the limiting side. The comparison `nums[left] < nums[right]` supplies exactly that guarantee. Any maximum on the left was only ever passed because some bar on the right was at least as tall at that time, and the right-side maximum only grows as the right pointer moves inward.

This is the same "shrink from the shorter side" principle behind Container With Most Water (LC 11): there too, the shorter wall is the one that limits the answer, so it is the one that gets discarded. When a quantity is capped by the minimum of two independent sides, two pointers that always advance from the limiting side are a recurring pattern worth recognizing.
