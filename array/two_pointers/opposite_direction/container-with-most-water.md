# Container With Most Water - LeetCode 11

---

## The Problem

You're given an array `height` where each value is the height of a vertical line drawn at that index on the x-axis. Pick **any two lines** — together with the x-axis they form a container. You want the container that holds the **most water**.

Two things decide how much water a container holds:

- **Width** — the distance between the two lines (`right - left`).
- **Height** — the *shorter* of the two lines, because water spills over the lower wall.

So the area for a pair of indices `(i, j)` is:

```
area = min(height[i], height[j]) * (j - i)
```

**Worked example**

```
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
index    0  1  2  3  4  5  6  7  8
```

Take the lines at index `1` (height 8) and index `8` (height 7):

- width = `8 - 1 = 7`
- height = `min(8, 7) = 7`
- area = `7 * 7 = 49`

No other pair does better, so the answer is **49**.

---

## Step 1: Brute Force

Try every possible pair of lines, compute the area, and keep the maximum.

```java
class Solution {
    public int maxArea(int[] height) {
        int maxArea = 0;
        int n = height.length;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int area = Math.min(height[i], height[j]) * (j - i);
                maxArea = Math.max(maxArea, area);
            }
        }
        return maxArea;
    }
}
```

**Time Complexity: O(n²)** — the outer loop picks `i`, the inner loop picks every `j > i`, giving n(n-1)/2 pairs. Each pair costs O(1) to evaluate.

**Space Complexity: O(1)** — only a few integer variables, no extra structures.

With `n` up to 10⁵ on LeetCode, n² is on the order of 10¹⁰ operations — this will time out.

---

## Step 2: Thinking Toward an Optimized Approach

Look at what the brute force is really doing: it evaluates *every* pair, but most of those pairs are obviously hopeless, and it has no way of knowing that without computing them. The waste is that each pair is treated independently, when in fact the pairs are related through width and height in a very structured way.

Start from the widest possible container: the first and last lines. This pair has the maximum possible width, so if we ever want to beat it with a narrower container, we will need a taller *limiting* wall. That reframes the problem. Every time we shrink the width by one, we lose something, and the only way to compensate is to gain height. So the question becomes: which end should we move inward?

Consider the two walls at the current pointers, and say the left one is shorter. The current area is limited by that left wall. Now ask what happens to any container that keeps this same left wall and pairs it with some line *closer in* on the right side:

- The width is strictly smaller than the current width.
- The height is still capped by the left wall (the shorter one) — it can never exceed it, even if the new right line is taller.

So every such container has width smaller and height no larger, which means its area is **guaranteed to be no better** than the one we just computed. The left wall has already given us the best it ever can. It is completely safe to discard it and never consider it again.

Now consider the opposite choice: moving the taller wall inward. The shorter wall still caps the height, and the width shrinks, so the area can only stay the same or get worse. That move can't help us, which is exactly why it's the wrong one.

This gives a clean rule: **always move the pointer at the shorter wall**, because it is the only move that has any chance of finding a taller limiting wall. Each step permanently eliminates one index from consideration, so the pointers sweep toward each other and we examine each index at most once. We have effectively pruned all the pairs the brute force wasted time on, without ever missing the optimal one, since the optimal pair can never be skipped by a move that is provably non-improving.

---

## My Optimized Solution

```java
class Solution {
    public int maxArea(int[] height) {
        int maxArea = 0;
        int area;
        int heigh, breadth;
        int left = 0, right = height.length - 1;
        while(left < right)
        {
            heigh = Math.min(height[left], height[right]);//as the container can hold the smallest height
            breadth = right - left;
            area = heigh * breadth;
            maxArea = Math.max(maxArea, area);
            //move the smallest height
            if(height[left] < height[right])
            left++;
            else
            right--;
        }
        return maxArea;
    }
}
```

### Walking through the logic

1. **Initialize** `maxArea = 0` and place two pointers at the extremes: `left = 0`, `right = height.length - 1`. This starts us at the widest container.
2. **Loop while `left < right`** — once the pointers meet, there is no valid pair left (a container needs two distinct lines).
3. **Compute the limiting height:** `heigh = Math.min(height[left], height[right])`. Water can only fill up to the shorter wall.
4. **Compute the width:** `breadth = right - left`.
5. **Compute the area** as `heigh * breadth` and update `maxArea` if this container beats the best seen so far.
6. **Move the pointer at the shorter wall:** if `height[left] < height[right]`, advance `left`; otherwise retreat `right`. This is the pruning step from Section 3 — the shorter wall can't do better with any narrower partner, so we discard it.
7. **Repeat** until the pointers cross, then return `maxArea`.

**Trace on `[1, 8, 6, 2, 5, 4, 8, 3, 7]`:**

| left | right | heigh | breadth | area | maxArea | move |
|------|-------|-------|---------|------|---------|------|
| 0 | 8 | 1 | 8 | 8 | 8 | left (1 < 7) |
| 1 | 8 | 7 | 7 | 49 | 49 | right (8 ≥ 7) |
| 1 | 7 | 3 | 6 | 18 | 49 | right (8 ≥ 3) |
| 1 | 6 | 8 | 5 | 40 | 49 | right (8 ≥ 8) |
| 1 | 5 | 4 | 4 | 16 | 49 | right (8 ≥ 4) |
| 1 | 4 | 5 | 3 | 15 | 49 | right (8 ≥ 5) |
| 1 | 3 | 2 | 2 | 4 | 49 | right (8 ≥ 2) |
| 1 | 2 | 6 | 1 | 6 | 49 | right (8 ≥ 6) |

Pointers meet → return **49**.

> **Notes (code left untouched):**
> - The solution is correct and optimal. The only things worth flagging are stylistic: `heigh` is a typo-style name for `height` (likely avoiding a clash with the parameter, so it's a reasonable choice), and the `if`/`else` has no braces, which works because each branch is a single statement but is easy to break if a second line is added later.
> - When `height[left] == height[right]`, the `else` branch moves `right`. That is fine — either pointer is safe to move on a tie, since neither wall can improve with a narrower partner.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (all pairs) | O(n²) | O(1) | Checks every pair; times out at n = 10⁵ |
| Two Pointers (mine) | O(n) | O(1) | Each index is visited at most once; prunes provably non-improving pairs |

---

## A Small Thing Worth Noting

This problem looks like it should need sorting or a clever data structure, but what makes two pointers valid here is a **monotonic exchange argument**, not any ordering of the array. The array is completely unsorted; the correctness comes from the fact that the area is bounded by `min(left, right) * width`, so discarding the shorter wall never discards the optimum.

That is the key thing to check whenever you reach for opposite-direction two pointers on an *unsorted* input: can you prove that moving a specific pointer eliminates candidates that are guaranteed to be no better? In sorted-array problems like Two Sum II, the sortedness gives you that guarantee for free. Here, the min-height cap plays the same role. The pattern is the same, the justification is different, and being able to articulate *why* the move is safe is what separates knowing this solution from being able to transfer it to new problems (for example, Trapping Rain Water uses the same "the shorter side is the bottleneck" reasoning).
