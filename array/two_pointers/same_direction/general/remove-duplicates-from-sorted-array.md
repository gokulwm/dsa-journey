# Remove Duplicates from Sorted Array

## 1. The Problem

You're given an array that is already sorted in non-decreasing order. You have to remove the duplicates **in place**, so that each distinct value appears exactly once at the front of the array, and return `k`, the number of distinct values. Whatever sits beyond index `k - 1` doesn't matter. You may not allocate a second array to hold the answer.

**Example**

```
Input:  nums = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
Output: 5, with nums = [0, 1, 2, 3, 4, _, _, _, _, _]
```

The five distinct values (0, 1, 2, 3, 4) get packed into the first five slots. The checker only looks at `nums[0..k-1]`, so the leftover tail can be anything.

---

## 2. Step 1: Brute Force

The most direct idea: scan the array, and every time an element equals its predecessor, delete it by shifting everything after it one step to the left.

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;          // logical length, shrinks as we delete
        int i = 1;
        while (i < n) {
            if (nums[i] == nums[i - 1]) {
                // delete nums[i] by shifting the suffix left
                for (int k = i; k < n - 1; k++) {
                    nums[k] = nums[k + 1];
                }
                n--;                  // one fewer element; do NOT advance i
            } else {
                i++;
            }
        }
        return n;
    }
}
```

**Time: O(n²).** In the worst case (all elements equal) there are about n deletions, and each one shifts up to n elements.
**Space: O(1).** Everything happens in place.

---

## 3. Step 2: Thinking Toward an Optimized Approach

The brute force is slow for one reason: it moves the same elements over and over. In an array of all 7s, the last element gets shifted left once for every single duplicate before it. All that shifting is paying to keep the array contiguous *at every moment*, but nobody asked for that. We only need the array to be correct once, at the end.

So the question becomes: can we place each surviving element directly into its final position, exactly once, instead of closing the gap after every deletion?

The sortedness is the key. Because equal values are adjacent, a value is "new" exactly when it differs from the last value we decided to keep. We never have to look back further than that one kept value, and we never have to search for earlier copies. That means we only need to track two things as we walk through the array: where the next kept value should be written, and which element we're currently inspecting. The reader finds new values, the writer records them. The writer can never overtake the reader, so writing into the array we're still reading from is safe: any slot the writer lands on has already been looked at. One pass, each element read once and written at most once.

---

## 4. My Optimized Solution

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length == 1) return 1;
        int i = 0, j = 0, count = 1;
        for(j = 0;j < nums.length;j++)
        {
            if(nums[i] != nums[j])
            {
                i+= 1;
                nums[i] = nums[j];
                count += 1;
            }
        }
        return count;
    }
}
```

**Walking through the logic**

1. `i` is the write pointer and always sits on the last unique value kept so far. `j` is the read pointer scanning the whole array. `count` tracks how many unique values have been kept.
2. The early return handles a single-element array, which trivially has one unique value.
3. For each `j`, compare `nums[j]` with `nums[i]`, the most recent unique value. If they're equal, `nums[j]` is a duplicate and we simply move on.
4. If they differ, we've found a new unique value. Advance `i` to the next free slot, copy `nums[j]` there, and increment `count`.
5. Because the array is sorted, comparing against the last kept value alone is enough to detect every duplicate.
6. After the loop, `nums[0..count-1]` holds the distinct values in order, and `count` is returned as `k`.

**Dry run on `[0, 0, 1, 1, 2]`**

| j | nums[j] | nums[i] | action | array | i | count |
|---|---------|---------|--------|-------|---|-------|
| 0 | 0 | 0 | equal, skip | [0,0,1,1,2] | 0 | 1 |
| 1 | 0 | 0 | equal, skip | [0,0,1,1,2] | 0 | 1 |
| 2 | 1 | 0 | new, write at 1 | [0,1,1,1,2] | 1 | 2 |
| 3 | 1 | 1 | equal, skip | [0,1,1,1,2] | 1 | 2 |
| 4 | 2 | 1 | new, write at 2 | [0,1,2,1,2] | 2 | 3 |

Result: `3`, with the first three slots holding `[0, 1, 2]`.



---

## 5. Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute force (shift on every duplicate) | O(n²) | O(1) | Re-moves the same elements repeatedly |
| Better (copy uniques into a `LinkedHashSet`/temp array, write back) | O(n) | O(n) | Linear time, but violates the spirit of "in place" |
| Optimized (two pointers, read/write) | O(n) | O(1) | Each element read once, written at most once |

---

## 6. A Small Thing Worth Noting

This is the template for a whole family of "compact an array in place" problems: LC 27 (Remove Element), LC 283 (Move Zeroes), and LC 80 (Remove Duplicates II) all use the same read-pointer/write-pointer skeleton. Only the condition for "keep this element" changes. Here it's `nums[j] != nums[i]` (differs from the last kept value). In LC 80 it becomes a comparison against the value two positions back in the kept region, which lets each value appear up to twice.

The invariant that makes it safe is worth remembering: the write pointer never passes the read pointer, so you never overwrite something you haven't read yet. The sortedness is what lets "compare to the last kept value" replace "search all previous values", which is the same property that would otherwise cost a hash set.
