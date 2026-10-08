# Move Zeroes - LeetCode 283

## 1. The Problem

You're given an integer array `nums`. Move every `0` to the end of the array, but keep the relative order of the non-zero elements exactly as it was. The catch: you must do this **in place**, without allocating a second array.

**Example**

```
Input:  nums = [0, 1, 0, 3, 12]
Output: nums = [1, 3, 12, 0, 0]
```

The non-zero values `1, 3, 12` appear in the same left-to-right order as before; the zeros are simply pushed to the back. Note that this is not a sorting problem. If you sorted the array you would get `[0, 0, 1, 3, 12]`, which puts the zeros at the front and breaks the "keep relative order" rule.

---

## 2. Step 1: Brute Force

The most direct idea: scan left to right, and every time you hit a zero, look ahead for the next non-zero element and swap it into place.

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length;
        for (int i = 0; i < n; i++) {
            if (nums[i] == 0) {
                // find the next non-zero element after i
                for (int j = i + 1; j < n; j++) {
                    if (nums[j] != 0) {
                        int temp = nums[i];
                        nums[i] = nums[j];
                        nums[j] = temp;
                        break;
                    }
                }
            }
        }
    }
}
```

**Time: O(n²).** For each zero at index `i`, the inner loop may scan the rest of the array to find a non-zero. With many zeros at the front (e.g. `[0, 0, 0, ..., 0, 5]`), each zero triggers a long scan before finding anything, so the total work is quadratic.

**Space: O(1).** Only a few index variables and a temp for swapping.

A simpler (and faster) non-in-place variant is to copy all non-zeros into a new array and pad with zeros. That gives O(n) time but O(n) extra space, which the problem explicitly asks us to avoid.

---

## 3. Step 2: Thinking Toward an Optimized Approach

Look at what the brute force keeps doing: every time it meets a zero, it goes hunting for the next non-zero element from scratch. But the scan that found the previous non-zero already walked over those positions. We are repeatedly re-examining the same stretch of the array, which is where the quadratic cost comes from.

The way out is to stop thinking about "moving zeros" and think about "placing non-zeros". The final array has a very simple shape: all the non-zero elements, in their original order, packed at the front, followed by however many zeros are needed to fill the rest. The number of zeros at the end is determined entirely by how many non-zeros we kept.

That suggests a different division of labour. Walk through the array once with a reader. Every time the reader sees a non-zero, it belongs in the next free slot of the packed front section, so keep a second index that marks exactly that next free slot. Because the reader never moves backward and the writer never overtakes the reader, no element is overwritten before it has been read, and the original order of the non-zeros is preserved automatically. Once the reader has finished, everything from the writer's position to the end is just leftover space, and it must all be zeros. So one pass sorts out the non-zeros, and a second, trivial pass fills the tail with zeros. Each element is touched a constant number of times, which is linear overall, and the only extra storage is one integer.

---

## 4. My Optimized Solution

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int place = 0;
        for(int i = 0;i < nums.length;i++)
        {
            if(nums[i] != 0)
            {
                nums[place] = nums[i];
                place += 1; 
            }
        }
        for(int i = place;i < nums.length;i++)
            nums[i] = 0;
    }
}
```

**Walking through the logic**

1. `place` is the write pointer. It always points to the next slot in the front section where a non-zero value should go. It starts at `0`.
2. The first loop uses `i` as the read pointer and visits every index once.
3. When `nums[i] != 0`, that value is copied to `nums[place]`, and `place` advances by one. When `nums[i] == 0`, nothing happens: the zero is simply skipped, and `place` stays behind `i`. This is how the gap between the two pointers grows, one skipped zero at a time.
4. Since `place <= i` at all times, writing to `nums[place]` can never destroy an element that hasn't been read yet. The invariant after each step is that `nums[0..place)` holds all non-zeros seen so far, in original order.
5. After the first loop, `place` equals the total count of non-zero elements. Everything from index `place` to the end is stale data (old values left over from the compaction).
6. The second loop overwrites every index from `place` to `nums.length - 1` with `0`, which produces exactly the required tail of zeros.

On `[0, 1, 0, 3, 12]`: after the first loop the array is `[1, 3, 12, 3, 12]` with `place = 3`; the second loop zeroes indices 3 and 4, giving `[1, 3, 12, 0, 0]`.


---

## 5. Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force (find next non-zero and swap) | O(n²) | O(1) | Re-scans the array for every zero |
| Better (copy non-zeros to a new array, pad zeros) | O(n) | O(n) | Linear, but violates the in-place requirement |
| Optimized (write pointer + zero fill) | O(n) | O(1) | Two linear passes, in place, order preserved |

---

## 6. A Small Thing Worth Noting

This is the **read/write pointer (compaction)** pattern, and it shows up far beyond this problem. Remove Element (LC 27), Remove Duplicates from Sorted Array (LC 26), and Remove Duplicates II (LC 80) are all the same skeleton: a reader scans everything, and a writer only advances when the reader finds something worth keeping. The only thing that changes between problems is the condition in the `if`.

It also hints at a useful way to design in-place algorithms: define the invariant for the "finished" prefix (here, `nums[0..place)` holds the kept elements in order) and make sure the writer can never run ahead of the reader. If that holds, overwriting is always safe, which is what lets you avoid extra memory.
