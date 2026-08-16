# Maximum Points You Can Obtain from Cards - LeetCode 1423

## The Problem

We are given an array `arr` of card points, and an integer `k`. We must pick exactly `k` cards total, but each pick has to come from **either the front or the back** of the array — never the middle. We want to maximize the sum of the picked cards.

**Example:**
```
arr = [1, 2, 3, 4, 5, 6, 1], k = 3
```
We must pick 3 cards, only from the front or the back. "Back `x` cards" means the last `x` elements of the array:
- 3 front, 0 back → `[1, 2, 3]` → sum = 6
- 2 front, 1 back → `[1, 2] + [1]` → sum = 4
- 1 front, 2 back → `[1] + [6, 1]` → sum = 8
- 0 front, 3 back → `[5, 6, 1]` → sum = 12 ✅ max

Answer: `12`, from taking 0 cards off the front and the last 3 cards off the back.

## Step 1: Brute Force

The direct way to think about it: try every possible split of "how many cards from the front, how many from the back" — from `0` front / `k` back all the way to `k` front / `0` back — and compute the sum for each split.

```java
class Solution {
    public int maxScore(int[] arr, int k) {
        int n = arr.length;
        int max = 0;

        for (int i = 0; i <= k; i++) {
            int sum = 0;
            // i cards from the front
            for (int f = 0; f < i; f++)
                sum += arr[f];
            // (k - i) cards from the back
            for (int b = n - (k - i); b < n; b++)
                sum += arr[b];

            max = Math.max(max, sum);
        }

        return max;
    }
}
```

**How it works:** the outer loop `i` decides how many cards are taken from the front (so `k - i` are taken from the back). The two inner loops sum up exactly those front and back cards for that split. Each split's total is compared against the best seen so far.

**Complexity:**
- Time: `O(k^2)` — for each of the `k + 1` splits, summing the front and back portions takes up to `O(k)` work.
- Space: `O(1)`

Just like the earlier sliding window problems, this recomputes almost the same sum again and again. Moving from split `i` to split `i + 1` only changes one card on each side — one card is removed from the back set and one card is added from the front set (or vice versa). That repeated work is exactly what a sliding window is meant to cut out.

## Step 2: Thinking Toward a Better Approach

Here's the reframe that unlocks this problem: instead of thinking about *which* cards get picked (split across two ends, which is awkward to slide over), think about **which cards get left behind.**

We pick `k` cards total, some from the front and the rest from the back. Whatever's left over has to be one unbroken chunk sitting somewhere in the array, because we only ever remove from the two ends inward. That leftover chunk always has the same fixed size, `n - k` — only its *position* shifts. Pick all `k` from the front → leftover sits at the very back. Pick all `k` from the back → leftover sits at the very front. Every split in between slides that leftover block across the array as one solid piece.

Since the total sum of the array never changes:

```
picked sum = total sum - leftover sum
```

So to **maximize** the picked sum, it's enough to **minimize** the leftover sum. And because the leftover block always has the same fixed size `n - k`, finding its minimum sum is a plain fixed-size sliding window — the same one-in-one-out mechanics as the vowels and threshold-average problems, just tracking a minimum instead of a maximum.

## Better Solution: Minimum Middle Window

```java
class Solution {
    public int maxScore(int[] arr, int k) {
        int n = arr.length;
        int total = 0;
        for (int x : arr)
            total += x;

        int windowSize = n - k;
        if (windowSize == 0) return total;

        int windowSum = 0;
        for (int i = 0; i < windowSize; i++)
            windowSum += arr[i];

        int minWindow = windowSum;

        for (int i = windowSize; i < n; i++) {
            windowSum += arr[i] - arr[i - windowSize];
            minWindow = Math.min(minWindow, windowSum);
        }

        return total - minWindow;
    }
}
```

**Walking through the logic (`arr = [1,2,3,4,5,6,1]`, `k = 3`):**
1. **Total sum of the array.** `1+2+3+4+5+6+1 = 22`.
2. **Leftover window size.** Since we pick `k = 3` cards, the leftover block has size `n - k = 7 - 3 = 4`.
3. **Sum the first window** — indices `[0, 3]` = `[1,2,3,4]` → `windowSum = 10`. This represents "leftover is the first 4 cards untouched," meaning we picked all 3 from the back. `minWindow = 10`.
4. **Slide to `[1, 4]`** = `[2,3,4,5]`: `windowSum = 10 + arr[4] - arr[0] = 10 + 5 - 1 = 14`. `minWindow` stays `10`.
5. **Slide to `[2, 5]`** = `[3,4,5,6]`: `windowSum = 14 + arr[5] - arr[1] = 14 + 6 - 2 = 18`. `minWindow` stays `10`.
6. **Slide to `[3, 6]`** = `[4,5,6,1]`: `windowSum = 18 + arr[6] - arr[2] = 18 + 1 - 3 = 16`. `minWindow` stays `10`.
7. **Loop ends.** The smallest leftover block found is `10` (the block `[1,2,3,4]` — meaning we picked the last 3 cards, `[5,6,1]`, which matches what we found by hand earlier).
8. **Return `total - minWindow` = `22 - 10 = 12`.**

**Complexity:**
- Time: `O(n)` — one pass to get the total, one pass to slide the window across the array.
- Space: `O(1)`

This is a real improvement over the brute force — no more recomputing sums from scratch for every split. But notice it still touches every single element of the array, even the ones far from either end that could never realistically matter much to a small `k`. That's the next thing to tighten up.

## Step 3: Thinking Toward the Optimal Approach

The min-middle-window solution scans the *whole array* (`O(n)`) to find where the leftover block should sit. But think about what we actually pick: only `k` cards, always from the two ends. The window of *picked* cards has size `k`, not `n - k` — and `k` can be much smaller than `n`.

So rather than sliding a window of size `n - k` across the whole array to find the minimum leftover, we can slide a window of size `k` directly over the **picked region** itself. Start by assuming all `k` picks come from the front. Then, one step at a time, trade the rightmost currently-picked card for the next card off the back of the array:

```
new_sum = old_sum + (new card picked up from the back) - (card dropped from the current pick)
```

This still checks every possible front/back split, but each step only costs `O(1)`, and there are only `k` splits to check — so the whole thing runs in `O(k)` instead of `O(n)`. When `k` is much smaller than `n`, this is the version that actually deserves to be called optimal.

## My Optimal Solution: Front/Back Swap

```java
class Solution {
    public int maxScore(int[] arr, int k) {
        int sum = 0, max;
        int l = k, r = arr.length;

        // Step 1: sum of taking all k cards from the front
        for(int i = 0;i < k;i++)
        sum += arr[i];
        max = sum;

        // Step 2: slide — trade one front card for one back card at a time
        while(l > 0)
        {
            l--;
            r--;
            sum = sum + arr[r] - arr[l];
            if(sum > max) max = sum;
        }

        return max;
    }
}
```

**Walking through the logic (`arr = [1,2,3,4,5,6,1]`, `k = 3`):**
1. **Start with the all-front pick.** Sum the first `k = 3` cards: `1 + 2 + 3 = 6`. This is `max` so far. `l = 3`, `r = 7` (length of array).
2. **First swap.** `l-- → 2`, `r-- → 6`. `sum = 6 + arr[6] - arr[2] = 6 + 1 - 3 = 4`. This is the split "2 from front, 1 from back" (`[1,2] + [1]`). `max` stays `6`.
3. **Second swap.** `l-- → 1`, `r-- → 5`. `sum = 4 + arr[5] - arr[1] = 4 + 6 - 2 = 8`. This is "1 from front, 2 from back" (`[1] + [6,1]`). `8 > 6`, so `max` becomes `8`.
4. **Third swap.** `l-- → 0`, `r-- → 4`. `sum = 8 + arr[4] - arr[0] = 8 + 5 - 1 = 12`. This is "0 from front, 3 from back" (`[5,6,1]`). `12 > 8`, so `max` becomes `12`.
5. **Loop ends** — `l` has reached `0`. Return `max = 12`. Same answer as the middle-window version, reached by touching only `k` elements instead of `n`.

Each step trades exactly one card: the leftmost card currently in our pick (`arr[l]`) is dropped, and one more card from the back of the array (`arr[r]`) is picked up instead — sliding the "pick window" from the front toward the back one card at a time.

**Complexity:**
- Time: `O(k)` — the initial sum takes `k` steps, and the sliding loop runs `k` more times.
- Space: `O(1)` — just a running sum, two pointers, and a max tracker.

## Brute Force vs Better vs Optimal

| | Brute Force | Better (Min Middle Window) | Optimal (Front/Back Swap) |
|---|---|---|---|
| Time | O(k^2) | O(n) | O(k) |
| Space | O(1) | O(1) | O(1) |
| Idea | Recompute front + back sum for every split | Minimize the sum of the untouched middle block | Reuse previous split's sum, just swap one card directly in the picked region |

## A Small Thing Worth Noting

This problem doesn't look like a classic sliding window at first glance, since what we're picking isn't a single contiguous run over the array — it's split across the front and back. The min-middle-window solution sidesteps that by sliding over the *unpicked* region instead, which is contiguous — a nice improvement over brute force, but it still costs `O(n)` because it walks the whole array. The front/back-swap solution goes one step further by sliding directly over the *picked* region, sized `k`, so it only ever does `k` steps of work regardless of how large the rest of the array is. Same underlying window idea both times — pick a contiguous region to slide over — just aimed at two different halves of the same picture, with the second choice being the tighter one.
