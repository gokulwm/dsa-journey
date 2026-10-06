# LeetCode 80: Remove Duplicates from Sorted Array II

## 1. The Problem

You're given an array that is already sorted in non-decreasing order. Modify it **in place** so that every distinct value appears **at most twice**, keeping the relative order, and return the new length `k`. Only the first `k` positions matter; whatever sits after them is ignored.

**Example**

```
Input:  nums = [1, 1, 1, 2, 2, 3]
Output: 5, with nums = [1, 1, 2, 2, 3, _]
```

The value `1` appears three times, so one copy has to go. `2` appears twice and `3` once, so both are fine. The catch is the in-place requirement: you can't just build a fresh array and return it, you have to rearrange `nums` itself using O(1) extra space.

---

## 2. Step 1: Brute Force

Scan left to right. Any time an element equals the one two positions before it, it must be a third (or later) copy, because the array is sorted. Delete it by shifting everything after it one step left.

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        int i = 2;
        while (i < n) {
            if (nums[i] == nums[i - 2]) {
                // third copy: delete it by shifting the tail left
                for (int x = i; x < n - 1; x++) {
                    nums[x] = nums[x + 1];
                }
                n--;
            } else {
                i++;
            }
        }
        return n;
    }
}
```

- **Time: O(n²).** Each deletion costs O(n) for the shift, and in the worst case (e.g. all elements equal) nearly every element gets deleted.
- **Space: O(1).** Everything happens inside `nums`.

---

## 3. Step 2: Thinking Toward an Optimized Approach

The brute force is slow for one reason: it pays to *close the gap* every time it deletes something. If ten elements get deleted, the same tail gets shifted ten separate times, even though every element in that tail ends up moving to a predictable final position anyway.

So the real question is: can we skip the shifting and write each surviving element directly into its final slot, exactly once? That needs a second pointer that marks "the next free slot in the answer" while a reader pointer races ahead. The reader is always at or ahead of the writer, so writing never clobbers anything the reader still needs.

The next question is how the reader decides what to keep. Since the array is sorted, equal values sit in contiguous runs. A run of length 1 contributes one element to the answer; a run of length 2 or more contributes exactly two. So instead of judging elements one by one, we can find where each run starts and ends, measure its length, and write `min(length, 2)` copies of that value. The reader makes a single pass, the writer makes a single pass, and no element is ever moved more than once. That collapses O(n²) into O(n) with no extra array.

---

## 4. My Optimized Solution

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int k = 0;
        int i = 0;
        int j;
        int res = 0;
        for(j = 1;j < nums.length;j++)
        {
            if(nums[i] != nums[j])
            {
                if(i+1 == j)
                {
                    nums[k] = nums[i];
                    k += 1;
                    res += 1;
                }
                else
                {
                    nums[k] = nums[i];
                    nums[k+1] = nums[i];
                    k += 2;
                    res += 2;
                }
                i = j;
            }
        }
        nums[k] = nums[i];
        res += 1;
        if(i < j-1)
        {
            nums[k+1] = nums[i];
            res += 1;
        }
        return res;
    }
}
```

**Walking through the logic**

1. `i` marks the **start of the current run** of equal values, `j` is the scanner moving right, and `k` is the **write position** for the final answer. `res` counts how many elements have been kept.
2. `j` advances while `nums[j] == nums[i]`, which means the run is still growing and nothing happens yet.
3. The moment `nums[j] != nums[i]`, the run `[i, j-1]` has ended and its length is `j - i`.
4. If `i + 1 == j`, the run has length 1, so write `nums[i]` once at `k`, then `k += 1` and `res += 1`.
5. Otherwise the run has length 2 or more, so write `nums[i]` twice at `k` and `k+1`, then `k += 2` and `res += 2`. This is where the extra copies get dropped.
6. Set `i = j` to begin tracking the new run.
7. The loop only flushes a run when it sees the *next* run begin, so the **last run is never flushed inside the loop**. After the loop, write it once, and write it a second time if `i < j - 1` (meaning its length is at least 2). Since `j == nums.length` at this point, `j - 1` is the last index.
8. Return `res`.

**Trace on `[1,1,1,2,2,3]`:** at `j=3` the run of `1`s (length 3) is flushed as two `1`s, so `k=2`. At `j=5` the run of `2`s (length 2) is flushed as two `2`s, so `k=4`. After the loop the final run `[3]` has length 1, so a single `3` is written at index 4 and `res=5`. Result: `[1,1,2,2,3,...]`.

---

## 5. Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force (shift on delete) | O(n²) | O(1) | Re-shifts the tail for every removed element |
| Better (copy into a new array) | O(n) | O(n) | Fast, but violates the in-place requirement |
| My Optimized (run detection + write pointer) | O(n) | O(1) | Single pass, each element written at most once |

---

## 6. A Small Thing Worth Noting

This solution works by explicitly detecting **run boundaries**, which is natural but hard-wired to "at most 2". The other well-known solution uses just a write pointer `k` and keeps `nums[j]` only if `k < 2 || nums[j] != nums[k-2]`. It has the same complexity, but it generalizes to "at most K duplicates" by changing a single number, since `nums[k-2]` becomes `nums[k-K]`. Both rely on the same underlying fact: because the array is sorted, a value that matches the element K slots behind the write position must be a (K+1)-th copy. Without sortedness, neither trick works, and you'd need a frequency map.
