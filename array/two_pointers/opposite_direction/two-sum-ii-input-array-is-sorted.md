# Two Sum II - Input Array Is Sorted - LeetCode 167

## The Problem

You're given an array of integers that is already sorted in non-decreasing order, and a `target`. Find the two numbers that add up to `target` and return their positions as a 1-indexed array `[index1, index2]` with `index1 < index2`.

Exactly one valid pair is guaranteed to exist, you can't use the same element twice, and you must use only constant extra space.

**Example:**

```
numbers = [2, 7, 11, 15], target = 9
```

`2 + 7 = 9`, and they sit at positions 1 and 2 (1-indexed), so the answer is `[1, 2]`.

The two details that shape everything: the array is **sorted**, and the space budget is **O(1)**. The first is a gift; the second rules out the usual hash map trick from the original Two Sum.

---

## Step 1: Brute Force

Try every pair `(i, j)` with `i < j` and return the first one that sums to `target`.

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int n = numbers.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = i + 1; j < n; j++) {
                if (numbers[i] + numbers[j] == target)
                    return new int[]{i + 1, j + 1};
            }
        }
        return new int[0];
    }
}
```

**Time: O(n²).** The outer loop runs about `n` times and the inner loop about `n - i` times each, giving `n(n-1)/2` pair checks in the worst case.

**Space: O(1).** Only a few index variables are used.

It meets the space limit but never uses the fact that the array is sorted, which is why it does far more work than necessary.

---

## Step 2: Thinking Toward an Optimized Approach

Think of every candidate pair `(i, j)` as a cell in an `n × n` grid, where the row is `i`, the column is `j`, and the value in the cell is `numbers[i] + numbers[j]`. The brute force walks this grid cell by cell. The question worth asking is: *when I check one cell, what am I actually learning?*

In an unsorted array, the answer is "just that one cell". A comparison tells you nothing about any other pair. But sortedness changes this completely, because the sum becomes **monotonic**: moving `j` rightward can only keep the sum the same or increase it, and moving `i` rightward does the same. Now a single comparison carries information about many cells at once.

Suppose we check the pair `(i, j)` and find the sum is **too small**. Then for this same `i`, every partner to the left of `j` is even smaller or equal, so it fails too. And `j` is already the largest partner still available to `i`. So `i` has no valid partner anywhere. It is not just "this pair failed", it's "the entire row for `i` is dead", and we can discard `i` permanently.

Symmetrically, if the sum is **too big**, then `j` paired with anything at or to the right of `i` is even bigger, and `i` is already the smallest partner `j` could have. So `j` has no valid partner, and we can discard `j` permanently.

Each comparison therefore eliminates a whole row or a whole column, not a single cell. That is the redundant work in the brute force: it re-examines rows and columns that a single comparison had already ruled out.

This tells us where to start. To have both moves available (advance the small end when the sum is too small, retreat the big end when it's too big), we begin at the two extremes, `i = 0` and `j = n - 1`. From there each step discards one index, so the two pointers walk toward each other and at most `n - 1` comparisons happen in total.

The correctness argument is worth stating precisely. Let `(p, q)` be the true answer. Claim: at all times `left ≤ p` and `right ≥ q`. If `left` were about to jump past `p` while `right` is still beyond `q`, then the current sum is `numbers[p] + numbers[right] ≥ numbers[p] + numbers[q] = target`, so we'd be in the "too big" or "exact" case, and `right` moves, not `left`. The same reasoning works symmetrically for `right`. So neither pointer can ever skip over the answer, and since they can't cross without passing it, they must land on it.

---

## My Optimized Solution

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;
        while(left < right)
        {
            int sum = numbers[left] + numbers[right];
            if(sum == target)
                return new int[]{left+1, right+1};
            else if(sum < target)
            left++;
            else
            right--;
        }
        return new int[0];
    }
}
```

**Notes on the code (nothing changed, just flagged):**

- `return new int[0];` is unreachable for valid inputs since the problem guarantees exactly one solution, but Java requires a return on every path, so it is needed to compile.
- `left++;` and `right--;` are not indented under their `if`/`else if`/`else`. It is legal without braces, but a future edit that adds a second statement would silently fall outside the branch. Braces would guard against that.
- `numbers[left] + numbers[right]` cannot overflow here since the constraints bound values to `[-1000, 1000]`. With unbounded `int` inputs, that sum would be a real concern.

### Walking through the logic

1. **Initialize two pointers** at opposite ends: `left = 0`, `right = numbers.length - 1`. These are the smallest and largest values, so the first sum is the midpoint of the possible range.
2. **Loop while `left < right`.** This guarantees two distinct elements are always being compared, which enforces "can't use the same element twice".
3. **Compute `sum`** of the two current elements. This is the only comparison made per iteration.
4. **If `sum == target`**, we're done. Return `left + 1` and `right + 1` because the problem wants 1-indexed positions.
5. **If `sum < target`**, the sum is too small, so `left++`. By the elimination argument, `numbers[left]` cannot pair with anything to its right, so it's safe to discard.
6. **Otherwise `sum > target`**, the sum is too big, so `right--`. `numbers[right]` cannot pair with anything to its left, so it's discarded.
7. **Each iteration discards exactly one index**, so the loop runs at most `n - 1` times.

**Trace on `numbers = [2, 7, 11, 15]`, `target = 9`:**

| Step | left | right | sum | Action |
|------|------|-------|-----|--------|
| 1 | 0 | 3 | 2 + 15 = 17 | 17 > 9, `right--` |
| 2 | 0 | 2 | 2 + 11 = 13 | 13 > 9, `right--` |
| 3 | 0 | 1 | 2 + 7 = 9 | match, return `[1, 2]` |

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (all pairs) | O(n²) | O(1) | Ignores sortedness |
| Better (binary search for the complement) | O(n log n) | O(1) | Uses sortedness, but only to speed up each lookup |
| Optimized (two pointers) | O(n) | O(1) | Uses sortedness to discard an index per comparison |

The middle tier: for each `numbers[i]`, binary search the remaining suffix for `target - numbers[i]`. That is `n` searches at `O(log n)` each. It exploits sortedness, but only locally, so it never reuses information between different `i` values. Two pointers does.

---

## A Small Thing Worth Noting

Two pointers works here because the sum function is **monotonic in each pointer independently**: increasing `left` never decreases the sum, and decreasing `right` never increases it. That is exactly what lets one comparison justify discarding an entire row or column. If the array weren't sorted, the same logic collapses, and you would fall back to a hash map, which is the standard solution to the original Two Sum but costs O(n) extra space, the exact thing this problem's constraint forbids.

The same elimination argument shows up in LeetCode 11 (Container With Most Water), where moving the shorter wall inward is safe because no container using that wall can do better, and it is the inner loop of 3Sum (LeetCode 15) once you fix the first element.
