# Permutation in String - LeetCode 567

## The Problem

We are given two strings `s1` and `s2`. We need to check whether `s2` contains a permutation of `s1` as a substring — meaning some contiguous chunk of `s2` has the exact same letters as `s1`, just possibly rearranged.

**Example:**
```
s1 = "ab", s2 = "eidbaooo"
```
Check every window of size 2 in `s2`:
- `"ei"` → not a permutation
- `"id"` → not a permutation
- `"db"` → not a permutation
- `"ba"` → same letters as "ab" → permutation ✅

Answer: `true` — "ba" at index 3 is a permutation of "ab".

## Step 1: Brute Force

Same first instinct as before: check every window of size `s1.length()` in `s2`, and see if that window is a rearrangement of `s1`. Sorting both and comparing is a simple way to check that.

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int n = s1.length(), m = s2.length();
        if (n > m) return false;

        char[] s1Arr = s1.toCharArray();
        Arrays.sort(s1Arr);
        String s1Sorted = new String(s1Arr);

        for (int i = 0; i <= m - n; i++) {
            String window = s2.substring(i, i + n);
            char[] wArr = window.toCharArray();
            Arrays.sort(wArr);
            String wSorted = new String(wArr);

            if (wSorted.equals(s1Sorted)) {
                return true;
            }
        }
        return false;
    }
}
```

**How it works:** `s1Sorted` is the sorted version of `s1`, computed once. Then for every window of `s2`, I cut out the substring, sort it too, and compare it against `s1Sorted`. The moment they match, a permutation has been found, so I return `true` right away.

**Complexity:**
- Time: `O((m - n) * n log n)` — for each of the `m - n + 1` windows, sorting costs `n log n`.
- Space: `O(n)` — for building the sorted strings.

Every window gets rebuilt and sorted from scratch, even though moving to the next window only changes one character on each end.

## Step 2: Thinking Toward an Optimized Approach

This is basically the same shape of problem as Find All Anagrams in a String — same fixed window, same idea that I only care about *letter counts*, not order. A frequency array of size 26 can compare counts in `O(26)`, which is effectively constant, instead of paying `O(n log n)` to sort every window.

And since the window only slides — it never resizes — I don't need to recount all `n` characters every time either. I can carry the frequency counts forward and just adjust them:
```
remove the character leaving on the left
add the character entering on the right
```

The one real difference from Find All Anagrams: this problem only asks "does a permutation exist," not "where are all of them." So the moment the two frequency arrays match, I can return `true` immediately instead of continuing to slide through the rest of the string.

## My Optimized Solution

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        if(s1.length() > s2.length()) return false;
        if(s1.length() == 1) return s2.contains(s1);
        int[] s1arr = new int[26];
        int[] s2arr = new int[26];
        int l = 0, r = s1.length();
        for(int i = 0;i < s1.length();i++)
        s1arr[s1.charAt(i) - 'a']+=1;
        for(int i = 0;i < s1.length();i++)
        s2arr[s2.charAt(i) - 'a']++;
        if(Arrays.equals(s1arr, s2arr))
        return true;
        while(r < s2.length())
        {
            s2arr[s2.charAt(l) - 'a'] -= 1;
            s2arr[s2.charAt(r) - 'a'] += 1;
            if(Arrays.equals(s1arr, s2arr)) return true;
            r++;
            l++;
        }
        return false;
    }
}
```

**Walking through the logic:**
1. **First edge case.** If `s1` is longer than `s2`, it can't possibly fit as a substring anywhere, so return `false` right away.
2. **Second edge case.** If `s1` is just a single character, checking for a permutation of it is the same as checking whether `s2` simply contains that character, so `s2.contains(s1)` handles it directly without needing the sliding window machinery.
3. **Two frequency arrays of size 26.** `s1arr` holds the fixed letter counts of `s1`. `s2arr` holds the letter counts of the *current window* in `s2`.
4. **Set up pointers.** `l` is the left edge of the window (starts at `0`), `r` starts at `s1.length()`, positioned just past the first window and ready to slide in next.
5. **Build the first window.** Count all characters of `s1` into `s1arr`, and count the first `s1.length()` characters of `s2` into `s2arr`.
6. **Check the first window.** If the two frequency arrays already match, a permutation exists right at the start, so return `true` immediately.
7. **Slide the window.** While `r` hasn't gone past the end of `s2`: remove `s2.charAt(l)` from `s2arr` (leaving the window), add `s2.charAt(r)` to `s2arr` (entering the window), and compare the two arrays. If they match, return `true` right away.
8. **Move both pointers forward** to shift the window ahead by one, and repeat.
9. **No match found.** If the loop finishes without ever matching, return `false`.

**Complexity:**
- Time: `O(m * 26)` → `O(m)` — each of the `m` characters is added and removed from the window at most once, and each comparison checks a fixed 26 slots.
- Space: `O(26)` → `O(1)` — two fixed-size frequency arrays.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O((m - n) * n log n) | O(m) |
| Space | O(n) | O(1) |
| Idea | Sort every window from scratch | Reuse previous window's counts, just adjust them |

## A Small Thing Worth Noting

This problem is almost identical in structure to Find All Anagrams in a String — same fixed window, same frequency-array comparison technique. The only real difference is what the question is asking for: "does at least one match exist" (so I return `true` the instant I find one) versus "give me every index where a match exists" (so I have to keep sliding through the entire string regardless).

That distinction is a good reminder that the *pattern* solving a problem and the *exact return behavior* around it are two separate decisions — recognizing the pattern gets me most of the way there, but I still need to read the question carefully to know whether to stop early or keep going.
