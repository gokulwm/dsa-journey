# Find All Anagrams in a String - LeetCode 438

## The Problem

We are given two strings `s` and `p`. We need to find every starting index in `s` where a substring of length `p.length()` is an anagram of `p` — meaning it has the exact same letters, in the same amounts, just possibly in a different order.

**Example:**
```
s = "cbaebabacd", p = "abc"
```
Check every window of size 3 in `s`:
- `"cba"` → same letters as "abc" → anagram ✅
- `"bae"` → not an anagram
- `"aeb"` → not an anagram
- `"eba"` → not an anagram
- `"bab"` → not an anagram
- `"aba"` → not an anagram
- `"bac"` → same letters as "abc" → anagram ✅

Answer: `[0, 6]` — anagrams start at index 0 and index 6.

## Step 1: Brute Force

The first instinct: check every window of size `p.length()` in `s`, and see if that window is a rearrangement of `p`. Sorting both the window and `p` and comparing them is a simple way to check that.

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();
        int n = s.length(), m = p.length();
        if (m > n) return result;

        char[] pArr = p.toCharArray();
        Arrays.sort(pArr);
        String pSorted = new String(pArr);

        for (int i = 0; i <= n - m; i++) {
            String window = s.substring(i, i + m);
            char[] wArr = window.toCharArray();
            Arrays.sort(wArr);
            String wSorted = new String(wArr);

            if (wSorted.equals(pSorted)) {
                result.add(i);
            }
        }
        return result;
    }
}
```

**How it works:** `pSorted` is the sorted version of `p`, computed once. Then for every window of `s`, I cut out the substring, sort it too, and compare the two sorted strings. If they match, every letter count lines up, so it's an anagram, and I record the starting index.

**Complexity:**
- Time: `O((n - m) * m log m)` — for each of the `n - m + 1` windows, sorting costs `m log m`.
- Space: `O(m)` — for building the sorted strings.

Every window gets rebuilt and sorted from scratch, even though sliding to the next window only changes one character on each end.

## Step 2: Thinking Toward an Optimized Approach

I don't actually need the letters in order — I only need to know whether the *letter counts* match. A frequency array of size 26 can answer that in `O(26)`, which is effectively constant, instead of paying `O(m log m)` to sort every single window.

And since the window only slides — it never resizes — I don't need to recount all `m` characters every time either. I can carry the frequency counts forward and just adjust them:
```
remove the character leaving on the left
add the character entering on the right
```

One thing worth being careful about here: I'm not using a `HashSet`, so there's no risk of `remove()` accidentally wiping out a letter that's still present elsewhere in the window. A frequency array just decrements a count, so it naturally handles duplicate letters correctly.

## My Optimized Solution

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        if(p.length() > s.length()) return new ArrayList<Integer>();
        int[] sarr = new int[26];
        int[] parr = new int[26];
        int l = 0, r = p.length();
        List<Integer> arr = new ArrayList<>();
        for(int i = 0;i < p.length();i++)
            parr[p.charAt(i) - 'a'] += 1;
        for(int i = 0;i < p.length();i++)
            sarr[s.charAt(i) - 'a'] += 1;
        if(Arrays.equals(parr, sarr))
            arr.add(l);
        while(r < s.length())
        {
            sarr[s.charAt(l) - 'a'] -= 1;
            sarr[s.charAt(r) - 'a'] += 1;
            l++;
            if(Arrays.equals(parr, sarr))
                arr.add(l);
            r++;
        }
        return arr;
    }
}
```

**Walking through the logic:**
1. **Edge case first.** If `p` is longer than `s`, no anagram can possibly fit, so return an empty list immediately.
2. **Two frequency arrays of size 26.** `parr` holds the fixed letter counts of `p`. `sarr` holds the letter counts of the *current window* in `s`.
3. **Set up pointers.** `l` is the left edge of the window (starts at `0`), `r` starts at `p.length()`, positioned just past the first window and ready to slide in next.
4. **Build the first window.** Count all characters of `p` into `parr`, and count the first `p.length()` characters of `s` into `sarr`.
5. **Check the first window.** If `parr` and `sarr` already match, index `0` is a valid start, so add it.
6. **Slide the window.** While `r` hasn't gone past the end of `s`: remove `s.charAt(l)` from `sarr` (leaving the window), add `s.charAt(r)` to `sarr` (entering the window), move `l` forward to the new start index, and compare the two arrays. If they match, this `l` is a valid starting index.
7. **Move `r` forward** to keep the window size fixed at `p.length()`, and repeat.
8. **Return the collected list** of all valid starting indices.

**Complexity:**
- Time: `O(n * 26)` → `O(n)` — each of the `n` characters is added and removed from the window at most once, and each comparison checks a fixed 26 slots.
- Space: `O(26)` → `O(1)` — two fixed-size frequency arrays.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O((n - m) * m log m) | O(n) |
| Space | O(m) | O(1) |
| Idea | Sort every window from scratch | Reuse previous window's counts, just adjust them |

## A Small Thing Worth Noting

`Arrays.equals(parr, sarr)` looks like a single operation, but it's actually comparing 26 elements every time it's called. Since 26 is a fixed constant no matter how big the input gets, this is still treated as `O(1)` overall — but it's a good reminder that "constant time" is often hiding a small fixed loop underneath.

This problem is also a nice example of why **counting beats sorting** when order was never part of the requirement — the brute force was solving a slightly stricter problem than what was actually being asked.
