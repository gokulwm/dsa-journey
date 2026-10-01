# 3Sum - LeetCode 15

## The Problem

Given an integer array `nums`, find every set of three *different positions* `i`, `j`, `k` whose values add up to `0`. The catch is that the answer must contain each distinct triplet (by value) only once, so `[-1, 0, 1]` and `[0, 1, -1]` count as the same triplet.

**Example:**

```
Input:  nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
```

- `-1 + -1 + 2 = 0` uses both `-1`s in the array.
- `-1 + 0 + 1 = 0` uses either `-1`, but it is still the same triplet by value, so it is reported once.

Finding triplets that sum to zero is the easy half. Reporting each one exactly once is where most of the difficulty lies.

---

## Step 1: Brute Force

Try every combination of three indices. Sorting first makes each triplet come out in a canonical order, so a `HashSet` can collapse duplicates like `[-1, 0, 1]` found via different `-1`s.

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Set<List<Integer>> set = new HashSet<>();
        Arrays.sort(nums);
        int n = nums.length;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                for (int k = j + 1; k < n; k++) {
                    if (nums[i] + nums[j] + nums[k] == 0) {
                        set.add(Arrays.asList(nums[i], nums[j], nums[k]));
                    }
                }
            }
        }
        return new ArrayList<>(set);
    }
}
```

**Time: O(n³).** Three nested loops each run up to `n` times, which gives about n³/6 combinations. Sorting costs O(n log n) and is dominated by this.

**Space: O(k)**, where `k` is the number of distinct triplets stored in the set (plus the sort's internal stack). The set is needed only because the loops have no way to avoid producing duplicates.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes work in the innermost loop. Once `i` and `j` are fixed, the value of `nums[k]` is completely determined: it has to be `-(nums[i] + nums[j])`. Yet the loop scans every remaining element to find one specific number. We are doing a linear search for something whose identity we already know.

The first fix is a lookup structure. If we put the candidates in a hash set, the third value can be checked in O(1), and the innermost loop disappears, giving O(n²) time. This is a real improvement, but it brings two problems. It costs O(n) extra memory, and duplicates are still painful: the same triplet can be reached through different pairs, so we are back to a set of lists, or to careful bookkeeping of which values have already been used.

The structure that removes both problems is **order**. After sorting, fix one number `nums[i]` and ask for two numbers in the rest of the array that sum to `-nums[i]`. In a sorted array, a pair sum is monotonic in both ends: moving the left pointer right can only increase the sum, and moving the right pointer left can only decrease it. So one pointer at each end can look at the current sum and know which direction to move, and each move permanently rules out a whole row of pairs that cannot work. A full pass over the rest of the array costs O(n) instead of O(n²), which brings the total to O(n²) with no hash set.

Sorting also solves the duplicate problem for free. Equal values sit next to each other, so skipping a number that equals its neighbour guarantees that the same value is never used twice in the same position of a triplet. The set of lists is no longer needed, because duplicates are never generated in the first place.

---

## My Optimized Solution

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        for(int i = 0;i < nums.length;i++)
        {
            if(i != 0 && nums[i-1] == nums[i])
                continue;
            int front = i + 1;
            int last = nums.length - 1;
            while(front < last)
            {
                if(nums[front] + nums[last] == -nums[i])
                {
                    res.add(Arrays.asList(nums[i], nums[front], nums[last]));
                    front++;
                    last--;
                    while(front < last && nums[last] == nums[last+1])
                        last--;
                    while(front < last && nums[front-1] == nums[front])
                        front++;
                }
                else if(nums[front] + nums[last] > -nums[i])
                {
                    last--;
                    while(front < last && nums[last] == nums[last+1])
                        last--;
                }
                else 
                {
                    front++;
                    while(front < last && nums[front-1] == nums[front])
                        front++;
                }
                
            }
        }
        return res;
    }
}
```

**Notes on the code (flagged, not changed):**

- The solution is correct, and it handles duplicates properly at all three positions.
- There is no early exit when `nums[i] > 0`. Since the array is sorted, once the fixed number is positive, no two numbers after it can bring the sum back to zero, so the loop could stop there. This doesn't change the O(n²) complexity, but it skips useless work on inputs with many positives.
- The duplicate-skipping `while` loops in the `>` and `<` branches are not needed for correctness. Duplicates only matter once a triplet has been recorded, and the match branch already handles that. They are harmless, and they just shorten the walk slightly.

### Walking through the logic

Using `nums = [-1, 0, 1, 2, -1, -4]`, which sorts to `[-4, -1, -1, 0, 1, 2]`:

1. **Sort the array.** This is what makes both the two-pointer movement and the duplicate skipping valid. Sorting costs O(n log n).
2. **Fix `nums[i]` as the first element of the triplet.** The task becomes: find two numbers to its right that sum to `-nums[i]`.
3. **Skip duplicate anchors.** `if(i != 0 && nums[i-1] == nums[i]) continue;` means a value is used as the first element only once. At `i = 2` the value `-1` equals `nums[1]`, so it is skipped, since every triplet starting with `-1` was already found from `i = 1`.
4. **Place two pointers.** `front = i + 1` and `last = nums.length - 1` span the entire remaining array.
5. **Compare the pair sum to the target `-nums[i]`.** For `i = 1` (value `-1`), the target is `1`.
   - **Equal:** record the triplet, then move both pointers inward. Here `front = 2` (`-1`) and `last = 5` (`2`) sum to `1`, so `[-1, -1, 2]` is recorded. Next, `front = 3` (`0`) and `last = 4` (`1`) sum to `1`, so `[-1, 0, 1]` is recorded.
   - **Sum too large:** `last--`, because only a smaller right value can reduce the sum.
   - **Sum too small:** `front++`, because only a larger left value can increase it.
6. **Skip duplicates after a match.** After recording a triplet, both pointers advance past any values equal to the ones just used (`nums[last] == nums[last+1]` and `nums[front-1] == nums[front]`). Without this, the same triplet would be added again by the next pair.
7. **Stop when the pointers meet.** `while(front < last)` ends the scan for this `i`, and the outer loop moves to the next anchor.
8. **Return `res`.** Because every skip step prevents duplicates at the source, no set is needed.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force (triple loop + `HashSet`) | O(n³) | O(k) | Duplicates removed after the fact by the set |
| Better (sort + fix `i`, hash lookup for the third value) | O(n²) | O(n) | Removes the innermost loop, but needs extra memory and clumsy dedup |
| Optimized: sort + two pointers (mine) | O(n²) | O(1) auxiliary (excluding sort and output) | Duplicates avoided by skipping equal neighbours |

`k` is the number of distinct triplets. The Better and Optimized rows have the same time complexity, so the two-pointer solution wins on memory and on how cleanly it handles duplicates, not on asymptotic speed.

---

## A Small Thing Worth Noting

Sorting does two jobs here, and they are easy to confuse. The first is **directional**: it makes the pair sum monotonic, so the pointers always know which way to move. The second is **structural**: it places equal values next to each other, so deduplication reduces to comparing a position with its neighbour. A hash set can replace the first job (as in the "Better" approach), but not the second.

This also shows how the pattern generalizes. 3Sum is "fix one element, then solve 2Sum on a sorted suffix." The same reduction extends to 4Sum (fix two elements, then two pointers) and in general to kSum, which costs O(n^(k-1)). Each extra level of nesting adds one more duplicate-skip check at that level. Whenever a problem asks for *unique combinations*, it is worth asking whether sorting can turn "avoid duplicates" into "skip equal neighbours."
