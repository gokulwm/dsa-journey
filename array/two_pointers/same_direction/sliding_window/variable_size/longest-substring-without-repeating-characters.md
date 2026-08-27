# Longest Substring Without Repeating Characters - LeetCode 3

## The Problem

Given a string `s`, find the length of the longest **substring** without repeating characters.

A substring must be contiguous — the characters have to sit next to each other in the original string, in order.

**Example**

```
Input:  s = "abcabcbb"
Output: 3
```

Walking through it window by window:

```
"a"        -> valid, length 1
"ab"       -> valid, length 2
"abc"      -> valid, length 3   <- longest so far
"abca"     -> 'a' repeats -> shrink from left until 'a' is gone -> "bca", length 3
"bcab"     -> 'b' repeats -> shrink -> "cab", length 3
"cabc"     -> 'c' repeats -> shrink -> "abc", length 3
"abcb"     -> 'b' repeats -> shrink -> "cb",  length 2
```

The window keeps growing and shrinking, but it never manages to beat length 3, so the answer is **3** (`"abc"`).

## Step 1: Brute Force

The most direct way to solve this: check **every possible substring**, and for each one, verify whether it has all unique characters. Keep track of the longest one that passes.

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                if (isUnique(s, i, j)) {
                    maxLen = Math.max(maxLen, j - i + 1);
                }
            }
        }
        return maxLen;
    }

    private boolean isUnique(String s, int start, int end) {
        Set<Character> seen = new HashSet<>();
        for (int k = start; k <= end; k++) {
            char c = s.charAt(k);
            if (seen.contains(c)) return false;
            seen.add(c);
        }
        return true;
    }
}
```

**Complexity:**
- Time: `O(n^3)` — `O(n^2)` substrings, and checking each one for uniqueness costs up to `O(n)`.
- Space: `O(n)` for the `seen` set used during each check.

## Step 2: Thinking Toward an Optimized Approach

The brute force wastes a huge amount of work re-checking overlapping parts of the string. When we move from checking substring `(i, j)` to `(i, j+1)`, almost the entire substring is the same — we're only adding one new character — but the brute force re-verifies uniqueness from scratch every single time.

The key realization: instead of restarting from every possible left boundary `i`, we can keep **one** window alive and only move its boundaries when necessary. If the window `[left, right]` currently has no repeating characters and we extend it to include `s[right+1]`, only one thing can go wrong — the new character might already exist somewhere in the current window. If it doesn't, the window is still valid and we've grown it for free. If it does, we don't need to throw away the whole window and restart — we just need to shrink from the `left` side, removing characters one at a time, until the duplicate is gone.

This turns the problem into a **variable-size sliding window**: expand `right` freely, and shrink `left` only when the window becomes invalid (a duplicate shows up). Since this is a "longest" problem, the rule is exactly the *expand-freely, shrink-only-when-broken* pattern — the window is never voluntarily shrunk while it's still valid. Each character enters the window once and leaves at most once, which is what brings the cost down from `O(n^3)` to `O(n)`.

## My Optimized Solution

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if(s.length() == 0 || s.length() == 1)
        return s.length();
        int maxlen = Integer.MIN_VALUE;
        int left = 0;
        int right = 1;
        HashMap<Character, Integer> set = new HashMap<>();
        set.put(s.charAt(left), 1);
        while(right < s.length())
        {
            char ch = s.charAt(right);
            while(set.containsKey(ch))
            {
                char lef = s.charAt(left);
                if(set.get(lef) == 1)
                    set.remove(lef);
                else
                    set.put(lef, set.get(lef) - 1);
                left += 1;
            }
            maxlen = Math.max(maxlen, right - left + 1);
            set.put(ch, set.getOrDefault(ch, 0) + 1);
            right += 1;
        }
        return maxlen;
      }
}
```

### Walking through the logic

1. **Edge case first.** If the string is empty or has a single character, the answer is just its length — there's no repeat possible, so return early.
2. **Seed the window.** `left` starts at index 0 and `right` starts at index 1. The map is seeded with `s.charAt(left)`, so the window effectively already contains one character before the main loop begins.
3. **Look at the incoming character.** For every step of `right`, grab `ch = s.charAt(right)` — this is the character we're trying to bring into the window.
4. **Shrink if it's a duplicate.** The inner `while(set.containsKey(ch))` loop keeps removing `s.charAt(left)` from the map and advancing `left`, one character at a time, until `ch` is no longer present in the map. This is the "fix the window" step — it only runs when `ch` would actually cause a repeat.
5. **Measure the window before adding `ch`.** At this point, the map represents exactly `[left, right-1]`, and we've just guaranteed `ch` isn't in there. So `[left, right]` — the window *including* `ch` — is valid, and its length is `right - left + 1`. `maxlen` is updated here.
6. **Add `ch` to the window.** Only after measuring do we insert `ch` into the map (`getOrDefault(ch, 0) + 1`), and move `right` forward.
7. **Repeat until `right` reaches the end of the string**, then return `maxlen`.

**A note on the `HashMap`:** because a duplicate is always resolved by shrinking *before* the character is re-added, no key in this map ever actually reaches a count greater than 1 during execution — the `if (set.get(lef) == 1) remove ... else decrement` branch is written defensively, but the `else` branch never triggers for this particular problem. Functionally, this map is being used as a `HashSet`. It still works perfectly correctly — it's just carrying more machinery than the problem strictly needs.

## Comparison

| Approach | Time Complexity | Space Complexity |
|---|---|---|
| Brute Force | O(n³) | O(n) |
| My Optimized Solution (Variable Sliding Window) | O(n) | O(min(n, alphabet size)) |

## A Small Thing Worth Noting

This problem is a clean example of the **longest-window** rule: expand `right` freely, and only shrink `left` when forced to. Notice that `maxlen` is updated *before* `ch` is inserted into the map — not after. That ordering isn't arbitrary: it reflects the fact that at the moment of measurement, the window `[left, right-1]` (tracked by the map) has already been proven not to contain `ch`, so `[left, right]` is provably valid even though `ch` hasn't been "recorded" yet. Measuring on provable validity rather than on the state of the data structure is a subtle but important habit — it's the same reasoning that, in the *minimum* window variant, flips around entirely: there, you measure *while* the window is still valid, before deliberately breaking it by shrinking further.
