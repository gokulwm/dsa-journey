# Two Sum Less Than K


## 1. The Problem

You are given an integer array `nums` and an integer `k`. Pick two elements at different indices `i < j` so that their sum is **as large as possible while still being strictly less than `k`**. Return that sum, or `-1` if no pair has a sum below `k`.

It is a "closest to the limit without crossing it" problem. A bigger sum is better, but only if it stays under `k`.

**Example**

```
nums = [34, 23, 1, 24, 75, 33, 54, 8], k = 60
```

- 34 + 24 = 58 (under 60, and the best we can do)
- 54 + 8 = 62 (too big, crosses 60)
- 23 + 34 = 57 (valid, but smaller than 58)

**Output: `58`**

If `nums = [10, 20, 30]` and `k = 15`, every pair sums to 30 or more, so the answer is `-1`.

---

## 2. Step 1: Brute Force

Check every pair, keep the largest sum that is below `k`.

```java
class Solution {
    public int twoSumLessThanK(int[] nums, int k) {
        int maxSum = -1;
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                int sum = nums[i] + nums[j];
                if (sum < k) {
                    maxSum = Math.max(maxSum, sum);
                }
            }
        }
        return maxSum;
    }
}
```

- **Time: O(n²).** The nested loops visit n(n-1)/2 pairs, and each check is O(1).
- **Space: O(1).** Only a few integer variables are used.

---

## 3. Step 2: Thinking Toward an Optimized Approach

The brute force treats every pair as an independent question, even though the pairs are related to each other. Take one element `a` and a candidate partner `b` that makes the sum too large. Every partner bigger than `b` will also be too large, but the brute force checks them all anyway. In the other direction, if `a + b` is under `k`, then a smaller partner can only give a smaller sum, so it can never beat `a + b`. The scan keeps looking at pairs whose outcome was already decided by an earlier comparison.

That kind of "a larger value always gives a larger sum" reasoning only works if the values are ordered, which is what sorting provides. Once the array is sorted, put one pointer at the smallest element and one at the largest, and look at the sum of the two. There are only two cases:

- If the sum is **too big** (`>= k`), the right element is too large to pair with the current left element. It is also too large to pair with anything further right of left, since those elements are even bigger. So the right element can be discarded for good, and `right` moves inward.
- If the sum is **valid** (`< k`), we record it. For this particular right element, the current left is the best partner seen so far, but a larger left might give an even bigger sum that is still valid. So `left` moves inward to look for a better one. Moving `right` instead would throw away a valid candidate that has not been fully explored.

Each step discards at least one element from consideration without losing any pair that could have been the answer. That turns O(n²) pair checks into a single O(n) sweep, after paying O(n log n) for the sort.

---

## 4. My Optimized Solution


```java
Arrays.sort(nums);
int left = 0; right = len-1;
int MaxSum = -1;
while (left < right)
{
    sum = nums[left] + nums[right];
    if (sum < target)
    {
        MaxSum = Math.max(sum, MaxSum);
        left++;
    }
    else right--;
}
```

**Walking through the logic**

1. **`Arrays.sort(nums)`** sorts the array ascending. This is what makes the pointer-moving decisions valid, because moving right always increases the sum and moving left always decreases it.
2. **`left = 0; right = len-1`** places the pointers at the two ends, so the first sum examined is the smallest element plus the largest.
3. **`while (left < right)`** keeps the two pointers on different indices, which matches the `i < j` requirement of the problem. When they meet, every useful pair has been covered.
4. **`sum = nums[left] + nums[right]`** is the candidate pair for this step.
5. **`if (sum < target)`** is the validity check. Here `target` is the `k` from the problem, and the comparison is strict.
6. **`MaxSum = Max(sum, MaxSum)`** records the best valid sum so far.
7. **`left++`** moves the smaller pointer up to try a bigger partner for the current `right`, hoping for a larger sum that is still under `k`.
8. **`else right--`** handles the case where the sum is `>= target`. The right element is too large to work with this left or any larger left, so it is dropped.

**Trace on the example** (sorted: `[1, 8, 23, 24, 33, 34, 54, 75]`, k = 60)

| left | right | sum | action | MaxSum |
|------|-------|-----|--------|--------|
| 1 | 75 | 76 | too big, `right--` | none yet |
| 1 | 54 | 55 | valid, `left++` | 55 |
| 8 | 54 | 62 | too big, `right--` | 55 |
| 8 | 34 | 42 | valid, `left++` | 55 |
| 23 | 34 | 57 | valid, `left++` | 57 |
| 24 | 34 | 58 | valid, `left++` | 58 |
| 33 | 34 | 67 | too big, `right--` | 58 |

After the last step the pointers meet and the loop ends with **58**.



## 5. Comparison Table

| Approach | Time Complexity | Space Complexity |
|----------|-----------------|------------------|
| Brute Force (all pairs) | O(n²) | O(1) |
| Sort + Two Pointers (mine) | O(n log n) | O(1) extra (sorting may use O(log n) stack space) |

The sort dominates the running time of the optimized solution. The two-pointer sweep itself is only O(n).

---

## 6. A Small Thing Worth Noting

Sorting is allowed here because the problem asks for a **sum**, not for the original indices. The indices `i` and `j` are never returned, so rearranging the array loses nothing. Problems like the classic Two Sum that return indices cannot use this trick directly.

Also, the O(n log n) is not the floor for this particular problem. The constraints bound every value by 1000, so a counting-sort variant that tallies values in an array of size 1001 and runs the same two-pointer logic over the values could run in O(n + range). Two-pointer after sorting is the general pattern. The bounded-value version is just a constraint-specific speed-up on top of it.
