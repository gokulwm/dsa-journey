# 4Sum - LeetCode 18

## The Problem

Given an integer array `nums` and an integer `target`, find every set of four numbers, taken from four *different positions*, whose sum equals `target`. As in 3Sum, each distinct quadruplet (by value) must be reported only once, so `[-2, -1, 1, 2]` and `[1, -1, 2, -2]` count as the same answer.

**Example:**

```
Input:  nums = [1, 0, -1, 0, -2, 2], target = 0
Output: [[-2, -1, 1, 2], [-2, 0, 0, 2], [-1, 0, 0, 1]]
```

- `-2 + -1 + 1 + 2 = 0`
- `-2 + 0 + 0 + 2 = 0` (the two `0`s are different positions, so both can be used)
- `-1 + 0 + 0 + 1 = 0`

There is one extra hazard compared with 3Sum. The values can reach 10⁹, so the sum of four of them can exceed what an `int` holds (about 2.1 × 10⁹), and a correct solution has to account for that.

---

## Step 1: Brute Force

Try every combination of four indices. Sorting first puts each quadruplet into a canonical order, so a `HashSet` can collapse repeats. The sum is computed as a `long` to avoid overflow.

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Set<List<Integer>> set = new HashSet<>();
        Arrays.sort(nums);
        int n = nums.length;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                for (int k = j + 1; k < n; k++) {
                    for (int l = k + 1; l < n; l++) {
                        long sum = (long) nums[i] + nums[j] + nums[k] + nums[l];
                        if (sum == target) {
                            set.add(Arrays.asList(nums[i], nums[j], nums[k], nums[l]));
                        }
                    }
                }
            }
        }
        return new ArrayList<>(set);
    }
}
```

**Time: O(n⁴).** Four nested loops, each running up to `n` times, which gives about n⁴/24 combinations. The O(n log n) sort is dominated by this.

**Space: O(k)**, where `k` is the number of distinct quadruplets stored in the set (plus the sort's internal stack). The set exists only because the loops produce the same quadruplet repeatedly.

---

## Step 2: Thinking Toward an Optimized Approach

The waste is the same as in 3Sum, one level deeper. Once `i`, `j`, and `k` are fixed, the fourth value is completely determined: it must be `target - nums[i] - nums[j] - nums[k]`. The innermost loop scans the rest of the array for one specific number whose identity is already known.

A hash lookup fixes that. Put the candidates in a set, check the fourth value in O(1), and the innermost loop disappears, giving O(n³). But it costs O(n) extra memory, and duplicates are again messy, because the same quadruplet is reachable through different triples and has to be filtered out afterward.

Order does better, and it works on the last two loops together instead of one. Fix the first two numbers `nums[i]` and `nums[j]`. The question left is whether two numbers in the remaining sorted suffix sum to `target - nums[i] - nums[j]`. This is the sorted pair-sum problem from 3Sum. A pointer at each end can compare the current sum with the goal and know which way to move, because moving `left` right can only raise the sum and moving `right` left can only lower it. Every move eliminates a whole row of pairs that cannot work, so the two innermost loops (O(n²) combined) collapse into a single O(n) scan. The total becomes n × n × n = O(n³) with no hash set.

Sorting also provides deduplication for free. Equal values are adjacent, so skipping a value that equals its neighbour guarantees each distinct value is used only once at each position of the quadruplet. With four positions there are four places to do this: the `i` loop, the `j` loop, and both pointers. Since duplicates are never generated, no set of lists is needed.

---

## My Optimized Solution

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        //Integer[] temp = new Integer[4];
        List<List<Integer>> ans = new ArrayList<>();
        for(int i=0;i < nums.length;i++)
        {
            if(i != 0 && nums[i-1] == nums[i])
                continue;
            for(int j = i+1;j < nums.length;j++)
            {
                if(j != i+1 && nums[j - 1] == nums[j])
                    continue;
                int left = j+1;
                int right = nums.length-1;
                while(left < right)
                {
                    Long sum = (long)nums[i] + (long)nums[j] + (long)nums[left] + (long)nums[right];
                    if(sum == target)
                    {
                        ans.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                        left++;
                        right--;
                        while(left < right && nums[left-1] == nums[left])
                            left++;
                        while(left < right && nums[right + 1] == nums[right])
                            right--;
                    }
                    else if(sum < target)
                    {
                        left++;
                        while(left < right && nums[left-1] == nums[left])
                            left++;
                    }
                    else
                    {
                        right--;
                        while(left < right && nums[right + 1] == nums[right])
                            right--;
                    }
                }
            }
        }
        return ans;
    }
}
```

**Notes on the code (flagged, not changed):**

- The solution is correct. Overflow is handled by casting each operand to `long`, and duplicates are skipped at all four positions.
- `Long sum` is the boxed wrapper type, so every iteration of the inner loop allocates or boxes a `Long`. The primitive `long sum` behaves identically here (the `==`, `<` comparisons with `int target` still work) and avoids that overhead. It affects constant factors only, not the complexity.
- The line `//Integer[] temp = new Integer[4];` is a leftover commented-out line and has no effect.
- There is no pruning. After sorting, if the smallest possible sum from position `i` (`nums[i]` plus the three next values) already exceeds `target`, or the largest possible sum is below it, the loop can be cut short. This helps on typical inputs but does not change the O(n³) worst case.

### Walking through the logic

Using `nums = [1, 0, -1, 0, -2, 2]` with `target = 0`, which sorts to `[-2, -1, 0, 0, 1, 2]`:

1. **Sort the array.** This is what makes both the pointer movement and the duplicate skipping valid.
2. **Fix `nums[i]` as the first element.** `if(i != 0 && nums[i-1] == nums[i]) continue;` ensures each value is used as the first element only once. At `i = 3`, the value `0` equals `nums[2]`, so it is skipped.
3. **Fix `nums[j]` as the second element, starting from `i+1`.** The duplicate check here is `j != i+1 && nums[j-1] == nums[j]`. Note that the "first allowed position" is `i+1`, not `0`. That exemption lets `nums[j]` equal `nums[i]` when it is the first choice for `j`, which is how a quadruplet like `[0, 0, 0, 0]` (from `nums = [0, 0, 0, 0]`, `target = 0`) gets found.
4. **Place two pointers.** `left = j+1` and `right = nums.length-1` span the rest of the array.
5. **Compute the four-way sum as a `long`.** Each operand is cast to `long` before adding, so the arithmetic never overflows.
6. **Compare with `target`.**
   - **Equal:** record the quadruplet, then move both pointers inward. For `i = 0` (`-2`), `j = 1` (`-1`): the first pair `left = 2` (`0`), `right = 5` (`2`) gives sum `-1`, which is too small, so `left` advances (and skips the duplicate `0`) to `left = 4` (`1`). Now `-2 + -1 + 1 + 2 = 0`, so `[-2, -1, 1, 2]` is recorded.
   - **Sum too small:** `left++`, because only a larger left value can raise the sum.
   - **Sum too large:** `right--`, because only a smaller right value can lower it.
7. **Skip duplicates for the pointers.** After each move, a pointer advances past any value equal to the one it just left (`nums[left-1] == nums[left]` and `nums[right+1] == nums[right]`). After a match this is required for correctness. After a non-match it only saves work.
8. **Stop when the pointers meet,** then move to the next `j`, then the next `i`.
9. **Return `ans`.** Every skip step prevents duplicates at the source, so no set is needed.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|---|---|---|---|
| Brute Force (four loops + `HashSet`) | O(n⁴) | O(k) | Duplicates removed after the fact by the set |
| Better (sort + fix `i, j, k`, hash lookup for the fourth value) | O(n³) | O(n) | Removes the innermost loop, but needs extra memory and clumsy dedup |
| Optimized: sort + fix `i, j` + two pointers (mine) | O(n³) | O(1) auxiliary (excluding sort and output) | Duplicates avoided by skipping equal neighbours |

`k` is the number of distinct quadruplets. The Better and Optimized rows share the same time complexity, so the two-pointer solution wins on memory and on how cleanly it handles duplicates, not on asymptotic speed.

---

## A Small Thing Worth Noting

The duplicate check in the `j` loop is `j != i+1`, not `j != 0`, and the difference carries the whole correctness argument. A duplicate check should compare a position against the **first element of its own range**, because the rule is "do not reuse a value at this position **within the same choice of earlier positions**." Inside the `j` loop, the earlier position is already fixed at `i`, so `j` is free to equal `nums[i]` the first time (at `i+1`), and only repeats after that are redundant. Writing `j != 0` would wrongly skip valid quadruplets: for `nums = [0, 0, 0, 0]` with `target = 0`, the pair `i = 0, j = 1` would be rejected because `nums[0] == nums[1]`, and the only valid answer would be lost.

The same logic applies at every level of the kSum reduction: each nesting level of the form "fix an element, then recurse on the suffix" needs a skip check whose boundary is the start of that level's suffix. The overflow cast is the other half of the lesson. The code is correct for an array of 10⁹ values only because the intermediate sum is widened *before* the additions happen, and casting after the `int` addition would be too late.
