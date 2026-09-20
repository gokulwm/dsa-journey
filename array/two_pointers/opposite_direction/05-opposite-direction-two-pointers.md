# Opposite-Direction Two Pointers (Converging Pointers)

## Idea

One pointer starts at the **left end** and the other at the **right end**, and they move **toward each other** until they meet or cross. At every step, the pair `(left, right)` is a candidate answer, and the pointers' movement is chosen so that each step **permanently eliminates** a set of candidates that can no longer be optimal (or valid).

The window `[left..right]` here is not a substring you are measuring — it is the **region of still-undecided indices**. Everything outside it has already been ruled out.

```
[ 1  2  4  7  11  15 ]      target = 15
  l                r         1 + 15 = 16  > 15  -> right--  (15 is too big for ANY partner ≥ 1)
  l            r             1 + 11 = 12  < 15  -> left++   (1 is too small for ANY partner ≤ 11)
     l         r             2 + 11 = 13  < 15  -> left++
        l      r             4 + 11 = 15  == 15 -> found
```

This replaces brute-force enumeration of all pairs (`O(n^2)`) with a single pass where every index is visited at most once, giving `O(n)`.

## The Idea Underneath (Why It Is Correct)

This is the part that transfers to new problems. Opposite-direction two pointers is not a "trick for sorted arrays" — it is an **elimination argument**:

> At each step, show that one endpoint **cannot be part of any better/valid pair** with any index still inside the region. If so, discard it safely.

- **Sorted pair-sum:** if `arr[l] + arr[r] < target`, then `arr[l]` paired with *any* index `j ≤ r` gives a sum `≤ arr[l] + arr[r] < target`. So `arr[l]` can never work. Discard it.
- **Bottleneck (Container With Most Water):** if `height[l] < height[r]`, then `l` paired with any closer index has smaller width and height still capped by `height[l]`. So `l` can never beat the current area. Discard it.
- **Symmetry (palindrome):** the pair `(l, r)` must match, and once checked, the pair carries no further information. Discard both.

The invariant to state every time: **"every pair involving an index outside `[left, right]` has already been ruled out."** If you cannot state what is being ruled out and why the move is safe, the pattern does not apply — even if the array happens to be sorted.

## When to use it

- The data is a **sequence with structure at both ends**: sorted, symmetric, or bounded by two limiting sides.
- The problem asks about a **pair** (or a pair nested inside a loop, as in 3Sum / 4Sum) rather than a contiguous window whose size is being measured.
- Signals:
  - "sorted array" + "find two numbers that ..." (sum / difference / closest to target)
  - "palindrome", "reverse", "mirror", "swap from both ends"
  - "maximize area / volume between two positions" (two walls)
  - "trapped water" (each position bounded by the max on its left and right)
  - "in-place, O(1) extra space" on an array that can be rearranged from the ends
  - Squares / absolute values of a sorted array (the extremes are the largest, the middle is the smallest)

If the input is **unsorted and you need original indices** (e.g. classic Two Sum returning indices), sorting destroys the information — use a HashMap instead.

## Core Invariant

- `left = 0`, `right = n - 1`; loop while `left < right`.
- At each step, examine the pair `(arr[left], arr[right])` and compute the quantity of interest.
- Move **exactly the pointer(s)** whose index has just been proven useless. Never move a pointer without being able to say why the discarded candidates cannot be better.
- The loop ends when the region is empty (`left >= right`); no pair remains undecided.

The only real design decision is the **movement rule**, and it depends on which of the variants below you are in.

## Four Variants — Classified by the Movement Rule

| | Target-Driven (sorted) | Symmetric Check | Bottleneck / Greedy | Extremes Merge |
|---|---|---|---|---|
| Typical problems | Two Sum II, 3Sum, 4Sum, Closest Sum | Valid Palindrome, Reverse String, Reverse Vowels | Container With Most Water, Trapping Rain Water | Squares of a Sorted Array |
| What must hold | Array is **sorted** (monotone in both ends) | Positions `i` and `n-1-i` are meant to correspond | An objective bounded by the **shorter/limiting** side | Ends are the extremes of the quantity |
| Move rule | Compare `arr[l] + arr[r]` with target; move the side that fixes the deficit | Move **both** inward after each match | Move the pointer at the **limiting** side | Take the larger end, move that pointer |
| What is eliminated | The endpoint that is too small / too large for every remaining partner | The already-verified pair | The limiting wall (it cannot improve with a narrower partner) | The end just consumed |
| Answer updated | On a hit (or tracking closest) | On first mismatch (early exit) | Every iteration (running max) | Every iteration (fill output) |
| Mental model | Thermostat — too low, raise the low end; too high, lower the high end | Zipper closing from both sides | Squeeze the weaker wall out | Peeling the outermost layer |

**Memory trick:**
- Sorted pair problems → *compare to target, move the side that can fix it.*
- Symmetry problems → *check the pair, move both.*
- Bottleneck problems → *the limiting side is the only side worth changing.*

### Variant 1 — Target-Driven (Sorted Array)

**Template**

```java
int left = 0, right = n - 1;

while (left < right) {
    long sum = (long) arr[left] + arr[right];   // long: avoid overflow

    if (sum == target) {
        // record answer / return
        left++;            // or right-- ; move both if collecting all unique pairs
        right--;
    } else if (sum < target) {
        left++;            // need a larger sum -> raise the small end
    } else {
        right--;           // need a smaller sum -> lower the large end
    }
}
```

**Extension — 3Sum / 4Sum:** fix one (or two) elements with an outer loop, run this template on the remainder. Complexity becomes `O(n^2)` (`O(n^3)` for 4Sum), and duplicates must be skipped at every level.

### Variant 2 — Symmetric Check

**Template**

```java
int left = 0, right = n - 1;

while (left < right) {
    if (!matches(arr[left], arr[right])) {
        return false;            // early exit on the first violation
    }
    left++;
    right--;                     // both move: the pair is fully resolved
}
return true;
```

Variants: skip irrelevant characters first (`while (left < right && !valid(arr[left])) left++;`), or swap instead of compare (reverse in place).

### Variant 3 — Bottleneck / Greedy Elimination

**Template**

```java
int left = 0, right = n - 1, best = 0;

while (left < right) {
    int limiting = Math.min(arr[left], arr[right]);
    best = Math.max(best, limiting * (right - left));   // update EVERY iteration

    if (arr[left] < arr[right]) left++;                  // move the limiting side
    else right--;
}
```

**Trapping Rain Water variant** — keep running maxima from each side and always process the side with the smaller maximum:

```java
int left = 0, right = n - 1, leftMax = 0, rightMax = 0, water = 0;

while (left < right) {
    if (height[left] < height[right]) {
        leftMax = Math.max(leftMax, height[left]);
        water += leftMax - height[left];
        left++;
    } else {
        rightMax = Math.max(rightMax, height[right]);
        water += rightMax - height[right];
        right--;
    }
}
```

### Variant 4 — Extremes Merge

**Template**

```java
int left = 0, right = n - 1;
int[] out = new int[n];

for (int pos = n - 1; pos >= 0; pos--) {          // fill output from the back
    if (Math.abs(arr[left]) > Math.abs(arr[right])) {
        out[pos] = arr[left] * arr[left];
        left++;
    } else {
        out[pos] = arr[right] * arr[right];
        right--;
    }
}
```

## Opposite Direction vs. Same Direction

| | Opposite Direction | Same Direction (Sliding Window) |
|---|---|---|
| Start | Both ends | Both at the start |
| Region tracked | Undecided indices `[left..right]`, shrinking | The current window `[left..right]`, moving right |
| Structure required | Sorted / symmetric / bounded by two sides | Contiguous subarray with an add/remove-able state |
| Typical target | A **pair** | A **subarray / substring** |
| Justification | Elimination argument (endpoint provably useless) | Monotone window validity (shrinking never hurts feasibility) |
| Auxiliary state | Usually none, `O(1)` | Often a set / map / counter |
| Ends when | `left >= right` | `right` reaches the end |

## Common Data Structures Used

| Structure | Used when |
|---|---|
| None (just the two indices) | The default; most of the pattern's power is that it needs no extra structure |
| Running maxima (`leftMax`, `rightMax`) | Trapping Rain Water and other "bounded by the best on each side" problems |
| Sorting beforehand (`Arrays.sort`) | The input is unsorted and you only need **values**, not original indices (3Sum, 4Sum, closest sum) |
| Output list + duplicate-skipping loops | Collecting all unique tuples (3Sum / 4Sum) |
| Output array filled from the back | Extremes-merge problems (Squares of Sorted Array, merging sorted structures) |

**A useful rule of thumb:** if you can sort the array and still answer the question with values alone, opposite-direction two pointers is usually available. If the answer must reference original positions, reach for a HashMap first.

## Complexity

| Aspect | Cost |
|---|---|
| Time (single scan) | `O(n)` — each iteration discards at least one index, so at most `n - 1` iterations |
| Time (with sorting) | `O(n log n)` — dominated by the sort, the scan itself stays `O(n)` |
| Time (3Sum / 4Sum) | `O(n^2)` / `O(n^3)` — outer loops times an `O(n)` inner scan |
| Space | `O(1)` for the scan itself; `O(n)` only if you need an output array, and `O(log n)` to `O(n)` hidden in the sort |

## Why This Beats Brute Force

Brute force treats every pair as independent and evaluates `n(n-1)/2` of them. Two pointers exploits the fact that pairs are **not** independent: because of sortedness (or the min-cap in a bottleneck problem), evaluating one pair tells you about a whole family of pairs at once. Each pointer move discards an entire row or column of the "pair matrix" without inspecting it. That is the same "reuse information, don't recompute" principle as the sliding window — applied here to pruning candidates rather than reusing window state.

## Pitfalls

- **Moving a pointer without a proof.** If you cannot say why the discarded index is useless, the algorithm may skip the true answer. This is the main failure mode on unsorted inputs.
- **Using it on unsorted data for sum problems.** Sum-driven movement relies on sortedness. Sort first (if indices are not needed) or use a HashMap.
- **Wrong loop condition.** `left < right` for pairs of distinct elements; `left <= right` only when a single element pairing with itself is valid (rare). Using `<=` by habit can pair an element with itself.
- **Integer overflow.** `arr[left] + arr[right]` can overflow `int` for large inputs — cast to `long`.
- **Duplicates in 3Sum / 4Sum.** Skip repeated values at the outer loop and, after recording a hit, skip repeats on both inner pointers. Skipping *before* checking the first element, or without the `i > 0` guard, silently drops valid answers.
- **Wrong move on a tie.** In bottleneck problems, on `arr[left] == arr[right]` either pointer is safe to move, but be consistent, and in Trapping Rain Water make sure the running-max comparison side matches the pointer you move.
- **Moving both pointers when you should move one.** In sum problems, moving both after a *non-match* skips valid pairs. Move both only after a hit (when collecting unique pairs) or a verified symmetric match.
- **Forgetting empty / single-element input.** With `n < 2` the loop body never runs — make sure the default return value is correct.

## Quick Recognition Checklist

1. Is the problem about a **pair** of positions (or a pair nested in a loop)? If it is about a contiguous window's size, think sliding window instead.
2. Is the array **sorted**, or can you sort it without losing needed information? → Target-driven variant.
3. Is there a **symmetry** to verify or restore (palindrome, reverse, mirror)? → Symmetric variant, move both.
4. Is the objective **capped by the shorter/limiting side** of the pair (area, water)? → Bottleneck variant, move the limiting side.
5. Are the **extremes** the interesting values (largest squares, biggest absolute values)? → Extremes-merge, consume from the ends.
6. Can you state, in one sentence, **which candidates each pointer move eliminates and why**? If not, the pattern does not apply.
