# 3Sum Closest - LeetCode 16

## The Problem

Given an integer array `nums` and an integer `target`, pick three numbers from three different positions so that their sum is as close to `target` as possible, and return that sum. Unlike 3Sum, you don't need the sum to hit the target exactly, and you return one number rather than a list of triplets. The problem guarantees exactly one closest sum.

**Example:**

```
Input:  nums = [-1, 2, 1, -4], target = 1
Output: 2
```

- `-1 + 2 + 1 = 2` is distance `1` from the target.
- `-4 + -1 + 2 = -3` is distance `4`, `-4 + 1 + 2 = -1` is distance `2`, and `-4 + -1 + 1 = -4` is distance `5`.
- So `2` is the closest achievable sum.

---

## Step 1: Brute Force

Try every combination of three indices, compute the sum, and keep the one with the smallest distance to `target`. Sorting isn't needed.

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int n = nums.length;
        int closest = nums[0] + nums[1] + nums[2];
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                for (int k = j + 1; k < n; k++) {
                    int sum = nums[i] + nums[j] + nums[k];
                    if (Math.abs(sum - target) < Math.abs(closest - target)) {
                        closest = sum;
                    }
                }
            }
        }
        return closest;
    }
}
```

**Time: O(n³).** Three nested loops, each up to `n` iterations, give about n³/6 combinations.

**Space: O(1).** Only a few integers are kept: the running best sum and the loop variables.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes work in the innermost loop. Once `i` and `j` are fixed, we are looking for a single `nums[k]` that brings the total as close to `target` as possible, which means the `nums[k]` nearest to `target - nums[i] - nums[j]`. That is a nearest-value search, and the loop does it by checking every remaining element one at a time.

Nearest-value search is exactly what sorting speeds up. If the array is sorted, a binary search finds the closest value to a goal in O(log n) instead of O(n). That turns the innermost loop into a logarithmic lookup and gives O(n² log n). It's a real improvement, but it still repeats work: every new `j` starts a fresh search from scratch, even though neighbouring searches have goals that are very close to one another.

The insight that removes the repetition is to stop treating `j` and `k` as two separate loops and treat them as a single **pair** whose sum we are steering toward a goal. Fix `i`, and the goal for the pair is `target - nums[i]`. Put one pointer at each end of the sorted suffix. If the current total is below `target`, then keeping `left` where it is and moving `right` further left can only make the sum smaller, so it moves away from the target. That means this `left` can never produce a better answer with any smaller `right`, and it can be discarded safely. The mirror argument holds when the total is at or above `target`: the current `right` cannot improve with any larger `left`, so `right` is discarded. Each step permanently rules out a whole row of candidate pairs, so one pass over the suffix costs O(n) and the whole algorithm is O(n²), with no extra memory.

One more point: this problem asks for a single sum, not every distinct triplet. So unlike 3Sum, duplicate handling here only affects speed and never correctness.

---

## My Optimized Solution

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int minVal = Integer.MAX_VALUE;
        int minDiff = Math.abs(minVal - target);
        if(minDiff == Integer.MIN_VALUE)
            minDiff = Integer.MAX_VALUE;
        Arrays.sort(nums);
        for(int i = 0;i < nums.length;i++)
        {
            if(i != 0 && nums[i-1] == nums[i])
            continue;
            int left = i+1;
            int right = nums.length-1;
            while(left < right)
            {
                int sum = nums[i] + nums[left] + nums[right];
                int diff = Math.abs(sum - target);
                if(diff < minDiff)
                {
                    minDiff = diff;
                    minVal = sum;
                }
                if(sum < target)
                {
                    left++;
                    while(left < right && nums[left - 1] == nums[left])
                        left++;
                }
                else
                {
                    right--;
                    while(left < right && nums[right] == nums[right+1])
                        right--;
                }
            }
        }
        return minVal;
    }
}
```

**Notes on the code (flagged, not changed):**

- The solution is correct, including the `minDiff` guard, which is not obvious at first glance (see the next point).
- The sentinel setup is more delicate than it looks. `Math.abs(Integer.MAX_VALUE - target)` overflows whenever `target` is negative. For most negative targets it wraps to a negative value whose absolute value is still a huge positive number, which works as "infinity". For `target = -1`, however, `MAX_VALUE + 1` wraps to exactly `Integer.MIN_VALUE`, and `Math.abs(Integer.MIN_VALUE)` returns `Integer.MIN_VALUE` itself (still negative), which would break the `diff < minDiff` comparison. Your `if(minDiff == Integer.MIN_VALUE)` line is what catches that case. A plain `int minDiff = Integer.MAX_VALUE;` would do the same job with no special case, since real differences are tiny under the problem's constraints.
- There is no early return when an exact match is found. When `sum == target` the difference is `0`, which can never be beaten, so the function could return `target` immediately. This is only a speed-up on lucky inputs.
- The unindented `continue;` under the duplicate check is cosmetic. It still belongs to the `if`.

### Walking through the logic

Using `nums = [-1, 2, 1, -4]` with `target = 1`, which sorts to `[-4, -1, 1, 2]`:

1. **Initialize the best answer.** `minVal` holds the best sum seen so far and `minDiff` its distance to `target`. They start at "infinity" values so that the first real sum always replaces them, and the guard handles the `target = -1` overflow case.
2. **Sort the array.** This makes the pair sum monotonic, so the pointers know which way to move.
3. **Fix `nums[i]` as the first element.** `if(i != 0 && nums[i-1] == nums[i]) continue;` skips a repeated anchor value, since a repeat would only recompute the same pair searches.
4. **Place two pointers.** `left = i+1` and `right = nums.length-1` span the rest of the array.
5. **Evaluate the current triplet.** Compute `sum` and its distance `diff = |sum - target|`. If `diff < minDiff`, record `minDiff = diff` and `minVal = sum`. For `i = 0` (`-4`): `left = 1` (`-1`), `right = 3` (`2`) gives sum `-3`, distance `4`, which becomes the best so far.
6. **Move one pointer based on which side of `target` the sum falls.**
   - **Sum below target:** `left++`, because this `left` can't do better with any smaller `right`. Continuing: `left = 2` (`1`), sum `-1`, distance `2`, a new best.
   - **Sum at or above target:** `right--`, because this `right` can't do better with any larger `left`. When `sum == target` this branch also runs, but `minDiff` is already `0` by then.
7. **Skip duplicates after each move.** Each pointer advances past values equal to the one it just left. This is purely a speed-up here, because skipped values would give the same sums.
8. **Repeat for the next `i`.** For `i = 1` (`-1`): `left = 2` (`1`), `right = 3` (`2`) gives sum `2`, distance `1`, the new best. Then `right--` ends the scan for this anchor.
9. **Return `minVal`.** After all anchors, `minVal = 2` is the closest achievable sum.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force (triple loop) | O(n³) | O(1) | Checks every triplet |
| Better (sort + fix `i, j`, binary search for the third value) | O(n² log n) | O(1) auxiliary | Innermost loop becomes a nearest-value search |
| Optimized: sort + fix `i` + two pointers (mine) | O(n²) | O(1) auxiliary (excluding sort) | Each pointer move discards a whole row of pairs |

The two-pointer solution is strictly better than the middle tier here, unlike 3Sum, where the hash approach matched it in time. The log factor disappears because the pointers share information between consecutive searches, while binary search starts over each time.

---

## A Small Thing Worth Noting

3Sum Closest looks like a small tweak of 3Sum, but the role of the duplicate-skipping lines changes completely. In 3Sum, skipping equal neighbours is a **correctness** requirement, because the output is a list of distinct triplets and every missed skip would produce a repeated answer. Here the output is a single number, and two identical values can only produce identical sums, so removing every duplicate-skipping line would still return the right answer, just a little slower on arrays with many repeats.

That distinction is worth asking about in every two-pointer problem: *does the problem ask me to return every combination, or one optimal value?* If it's every combination, deduplication belongs to the logic. If it's one value, the same lines are only an optimization, and you can include or drop them without worrying about correctness.
