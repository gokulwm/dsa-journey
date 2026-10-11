# Merge Sorted Array - LeetCode 88

## 1. The Problem

You're given two sorted arrays, `nums1` and `nums2`. `nums1` has length `m + n`: its first `m` slots hold real values and the last `n` slots are zero placeholders, reserved for the elements of `nums2`. Merge `nums2` into `nums1` so that `nums1` ends up as one sorted array. The result must be stored **inside `nums1` itself**, so you can't return a new array.

Example:

```
nums1 = [1, 2, 3, 0, 0, 0],  m = 3
nums2 = [2, 5, 6],           n = 3

nums1 after merge = [1, 2, 2, 3, 5, 6]
```

The trailing zeros in `nums1` aren't data. They're empty space that the merged result will grow into.

---

## 2. Step 1: Brute Force

Copy every element of `nums2` into the empty tail of `nums1`, then sort the whole array.

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        for (int j = 0; j < n; j++) {
            nums1[m + j] = nums2[j];
        }
        Arrays.sort(nums1);
    }
}
```

**Time: O((m + n) log(m + n)).** The copy is O(n), but the sort dominates. Sorting a length-`(m + n)` array costs `(m + n) log(m + n)` comparisons.

**Space: O(1) extra** (ignoring the stack/internal buffers the sort itself may use, which is O(log(m + n)) for Java's dual-pivot quicksort on primitives).

It's correct, but it ignores the most important fact in the problem: both halves are *already sorted*.

---

## 3. Step 2: Thinking Toward an Optimized Approach

The brute force throws away information. By the time we call `Arrays.sort`, we've mashed two perfectly ordered sequences together and then asked a general-purpose sorter to rediscover the order from scratch. Sorting from scratch costs `log` factors that we shouldn't have to pay, because merging two sorted lists is a linear operation. You only ever need to compare the *front* of each list and take the smaller one.

The textbook merge walks both arrays from the front. Here that breaks down: if we write the smallest element into `nums1[0]`, we overwrite a value in `nums1` that hasn't been consumed yet. The fix people reach for is a temporary copy of `nums1`, which buys correctness at the cost of O(m) extra space.

The better question is: where in `nums1` is there free space that's safe to write into? The answer is the *back*. The last `n` slots are empty, so nothing there can be destroyed. That suggests flipping the direction of the merge. Instead of asking "what is the smallest remaining element?" and placing it at the front, ask "what is the *largest* remaining element?" and place it at the back. The largest of everything left is always the larger of the two tails, so it's still just one comparison per step. And because we're filling from the back, we only ever overwrite slots that are either empty or already consumed.

So: three pointers, one reading from the end of each array's real data, one writing from the end of the full buffer. Linear time, no extra space.

---

## 4. My Optimized Solution

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1;
        int j = n - 1;
        int k = m+n - 1;
        while(i >= 0 && j >= 0)
        {
            if(nums1[i] > nums2[j])
                nums1[k] = nums1[i--];
            else
                nums1[k] = nums2[j--];
            k--;
        }
        while(j >= 0)
            nums1[k--] = nums2[j--];
    }
}
```

### Walking through the logic

1. **Set up three pointers.** `i = m - 1` points at the last real element of `nums1`, `j = n - 1` points at the last element of `nums2`, and `k = m + n - 1` points at the last slot of `nums1`, which is where the next-largest element will be written.
2. **Main loop: while both arrays still have unprocessed elements** (`i >= 0 && j >= 0`), compare `nums1[i]` with `nums2[j]`.
3. **Place the larger one at `k`.** If `nums1[i]` is larger, move it to `nums1[k]` and decrement `i`. Otherwise take `nums2[j]` and decrement `j`. Either way, decrement `k` because that slot is now final.
4. **Leftovers from `nums2`.** If the loop ends because `i` ran out first, any remaining elements of `nums2` are all smaller than everything already placed, so copy them straight into the front slots.
5. **No cleanup needed for `nums1`.** If instead `j` runs out first, the remaining `nums1[0..i]` elements are already in their correct final positions, so there's nothing left to do.

Trace on `nums1 = [1,2,3,0,0,0]`, `nums2 = [2,5,6]`:

| Step | i | j | k | Compare | Write | nums1 |
|------|---|---|---|---------|-------|-------|
| 1 | 2 | 2 | 5 | 3 vs 6 | 6 (from nums2) | [1,2,3,0,0,**6**] |
| 2 | 2 | 1 | 4 | 3 vs 5 | 5 (from nums2) | [1,2,3,0,**5**,6] |
| 3 | 2 | 0 | 3 | 3 vs 2 | 3 (from nums1) | [1,2,3,**3**,5,6] |
| 4 | 1 | 0 | 2 | 2 vs 2 | 2 (from nums2) | [1,2,**2**,3,5,6] |

Now `j < 0`, so the loop ends and the second `while` does nothing. Final: `[1, 2, 2, 3, 5, 6]`.

---

## 5. Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute force (copy + sort) | O((m + n) log(m + n)) | O(1) | Ignores that inputs are already sorted |
| Better (forward merge with a copy of `nums1`) | O(m + n) | O(m) | Linear, but needs a temporary array to avoid overwriting |
| Optimal (my solution: merge from the back) | O(m + n) | O(1) | Linear and in-place |

---

## 6. A Small Thing Worth Noting

Why can't the write pointer `k` ever overwrite an unread element of `nums1`? Because of an invariant: `k = i + j + 1` holds throughout, so since `j >= -1`, we always have `k >= i`. The write position stays at or ahead of the read position in `nums1`. The only way `k` could land on an unread `nums1` element is if `k == i`, and that only happens when `j == -1`, meaning `nums2` is exhausted. At that point the remaining elements are already where they belong.

This is the same idea behind a lot of in-place array tricks: **if free space exists at one end, fill from that end.** Whenever a forward pass would clobber data you still need, check whether running the same logic in reverse turns the free space into a safe write zone.
