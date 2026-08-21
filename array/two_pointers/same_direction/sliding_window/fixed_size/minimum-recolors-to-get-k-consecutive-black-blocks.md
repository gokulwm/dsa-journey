# Minimum Recolors to Get K Consecutive Black Blocks - LeetCode 2379

## The Problem

We are given a string `blocks` made up only of `'W'` (white) and `'B'` (black) characters, and a number `k`. We need to find the minimum number of white blocks that must be recolored to black so that some window of `k` consecutive blocks is entirely black.

**Example:**
```
blocks = "WBBWWBBWBW", k = 7
```
Check every window of size 7:
- `"WBBWWBB"` → 3 white blocks → recolor 3
- `"BBWWBBW"` → 3 white blocks → recolor 3
- `"BWWBBWB"` → 4 white blocks → recolor 4
- `"WWBBWBW"` → 4 white blocks → recolor 4

Answer: `3` — the minimum white blocks in any window of size 7 is 3, so that's the fewest recolors needed.

## Step 1: Brute Force

The first instinct: check every window of size `k`, count how many white blocks are in it, and keep track of the smallest count seen. That smallest count is the answer, since each white block needs exactly one recolor.

```java
class Solution {
    public int minimumRecolors(String blocks, int k) {
        int n = blocks.length();
        int minRecolors = Integer.MAX_VALUE;

        for (int i = 0; i <= n - k; i++) {
            int white = 0;
            for (int j = i; j < i + k; j++) {
                if (blocks.charAt(j) == 'W') white++;
            }
            minRecolors = Math.min(minRecolors, white);
        }
        return minRecolors;
    }
}
```

**How it works:** the outer loop `i` fixes the starting index of the window, and the inner loop `j` counts how many `'W'` characters fall inside that window. After counting, I compare it against the smallest count found so far and keep the minimum.

**Complexity:**
- Time: `O(n * k)` — for each of the `n - k + 1` windows, the inner loop does `k` work to count white blocks.
- Space: `O(1)`

Every window gets recounted from scratch, even though sliding to the next window only changes one character on each end.

## Step 2: Thinking Toward an Optimized Approach

The window size `k` is fixed — it only slides, it never resizes. That means I don't need to recount white blocks for every window. I can carry the white count forward and just adjust it as the window slides:
```
if the block leaving on the left was white, subtract one
if the block entering on the right is white, add one
```

This turns the inner `O(k)` counting work into `O(1)` per step, so the whole thing becomes a single pass sliding window, tracking the minimum white count seen along the way.

## My Optimized Solution

```java
class Solution {
    public int minimumRecolors(String bl, int k) {
        int white = 0, black = 0, minch = 0;
        int l = 0, r = k;
        for(int i = 0;i < k;i++)
            if(bl.charAt(i) == 'W') white += 1;
        minch = white;
        while(r < bl.length())
        {
            if(bl.charAt(l) == 'W') white -= 1;
            if(bl.charAt(r) == 'W') white += 1;
            if(minch > white) minch = white;
            l++;
            r++;
        }
        return minch;
    }
}
```

**Walking through the logic:**
1. **Set up pointers.** `l` is the left edge of the window (starts at `0`), `r` starts at `k`, positioned just past the first window and ready to slide in next.
2. **Count the first window.** Loop through the first `k` characters and count how many are `'W'`, storing that in `white`.
3. **Initialize `minch`** with this first window's white count, since it's the only window checked so far.
4. **Slide the window.** While `r` hasn't gone past the end of the string:
   - If the block leaving on the left (`bl.charAt(l)`) is white, decrement `white`.
   - If the block entering on the right (`bl.charAt(r)`) is white, increment `white`.
   - Compare `white` against `minch` and keep the smaller one.
   - Move both `l` and `r` forward to shift the window by one.
5. **Return `minch`** — the fewest white blocks found in any window, which is exactly the minimum number of recolors needed.

**Complexity:**
- Time: `O(n)` — each character enters and leaves the running count exactly once.
- Space: `O(1)` — just a running count, two pointers, and a running minimum.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O(n * k) | O(n) |
| Space | O(1) | O(1) |
| Idea | Recount white blocks for every window | Reuse previous window's count, just adjust it |

## A Small Thing Worth Noting

The `black` variable in the optimized solution is declared but never actually used — it's a leftover from an earlier line of thinking (probably tracking black blocks directly at some point) that didn't end up being needed once the logic settled on tracking `white` instead. It doesn't affect correctness or complexity, but it's a small reminder to clean up unused variables once the final approach is locked in.

This problem is also a good example of a min-window variant of the same fixed sliding window pattern used in problems like Maximum Number of Vowels in a Substring — instead of tracking a running count and looking for the *maximum*, here the running count (white blocks) is something to *minimize* instead.
