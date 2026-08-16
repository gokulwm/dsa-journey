# Maximum Number of Vowels in a Substring of Given Length - LeetCode 1456

## The Problem

We are given a string `s` and an integer `k`. We need to find the maximum number of vowels (`a`, `e`, `i`, `o`, `u`) that can be found in any substring of length `k`.

**Example:**
```
s = "abciiidef", k = 3
```
Check every window of 3 consecutive characters:
- `abc` → 1 vowel (a)
- `bci` → 1 vowel (i)
- `cii` → 2 vowels (i, i)
- `iii` → 3 vowels (i, i, i) ✅ max so far
- `iid` → 2 vowels (i, i)
- `ide` → 2 vowels (i, e)
- `def` → 1 vowel (e)

Answer: `3`, coming from the window `"iii"`.

## Step 1: Brute Force

Same first instinct as always — check every window of size `k` directly, count the vowels in it from scratch, and keep the best count seen so far.

```java
class Solution {
    public int maxVowels(String s, int k) {
        int maxCount = 0;

        for (int i = 0; i <= s.length() - k; i++) {
            int count = 0;
            for (int j = i; j < i + k; j++) {
                char c = s.charAt(j);
                if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u')
                    count++;
            }
            maxCount = Math.max(maxCount, count);
        }

        return maxCount;
    }
}
```

**How it works:** the outer loop `i` fixes the starting index of the window, and the inner loop `j` scans all `k` characters of that window, counting vowels one by one. Once the window's count is known, it's compared against the best one found so far.

**Complexity:**
- Time: `O(n * k)` — for each of the `n - k + 1` windows, the inner loop re-scans `k` characters.
- Space: `O(1)`

Same problem as before — consecutive windows overlap almost completely, but the inner loop recounts the whole thing from scratch every single time.

## Step 2: Thinking Toward an Optimized Approach

`k` is fixed here too, so the window only slides, it never resizes. When the window moves from `[i, i+k-1]` to `[i+1, i+k]`, only one character leaves (index `i`) and only one character enters (index `i+k`). Everything else in the middle stays exactly the same.

So instead of recounting vowels for the whole window every time, I can carry the count forward:
```
new_count = old_count - (1 if the leaving character was a vowel else 0) + (1 if the entering character is a vowel else 0)
```

This turns the repeated `O(k)` rescanning into `O(1)` work per step — one full pass through the string.

## My Optimized Solution

```java
class Solution {
    public int maxVowels(String s, int k) {
        int result  = 0, count = 0;
        int l = 0, r = k;

        // Step 1: vowel count of the very first window
        for(int i = 0;i < k;i++)
            if(s.charAt(i) == 'a' || s.charAt(i) == 'e' ||s.charAt(i) == 'o' || s.charAt(i) == 'i' || s.charAt(i) == 'u')
                count++;

        result = count;

        // Step 2: slide the window across the rest of the string
        while(r < s.length())
        {
            if(s.charAt(l) == 'a' || s.charAt(l) == 'e' ||s.charAt(l) == 'o' || s.charAt(l) == 'i' || s.charAt(l) == 'u')
                count--;
            if(s.charAt(r) == 'a' || s.charAt(r) == 'e' ||s.charAt(r) == 'o' || s.charAt(r) == 'i' || s.charAt(r) == 'u')
                count++;

            if(count > result) result = count;

            r++;
            l++;
        }

        return result;
    }
}
```

**Walking through the logic:**
1. **Set up the first window.** Count vowels in the first `k` characters and store it as `count`. This becomes `result` right away, since it's window #1 and hasn't been compared to anything yet.
2. **Two pointers, `l` and `r`.** `l` is the character about to leave the window, `r` is the character about to enter it. `r` starts at `k` since the first window already covers indices `0` to `k-1`.
3. **Slide one step at a time.** Decrement `count` if `s.charAt(l)` was a vowel (it's leaving), increment `count` if `s.charAt(r)` is a vowel (it's entering), then check if this beats `result`.
4. **Move both pointers forward** after checking, so the window shifts by exactly one position for the next iteration.
5. **Return `result`** — the maximum vowel count found across all windows.

**Complexity:**
- Time: `O(n)` — each character enters and leaves the running count exactly once.
- Space: `O(1)` — just a running count, two pointers, and a result tracker.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O(n * k) | O(n) |
| Space | O(1) | O(1) |
| Idea | Recount vowels for every window | Reuse previous window's count, just adjust it |

## A Small Thing Worth Noting

This is the same fixed-size sliding window skeleton as Maximum Average Subarray I and Sub-arrays of Size K and Average ≥ Threshold — `l` and `r` tracking the boundary, one value leaving and one entering per step. The only thing that changes problem to problem is *what* is being tracked inside the window: there it was a running sum, here it's a running vowel count. Once this skeleton clicks, it applies directly regardless of what condition is being checked inside.
