# Longest Substring with At Most 2 Distinct Characters - LeetCode 159


## The Problem

Given a string, find the length of the longest substring that contains **at most 2 distinct characters**.

Worked example: `s = "eceba"`

- `"ec"` → 2 distinct chars (`e`, `c`) → valid, length 2
- `"ece"` → 2 distinct chars (`e`, `c`) → valid, length 3
- `"eceb"` → 3 distinct chars (`e`, `c`, `b`) → invalid
- `"ceb"` → 3 distinct chars → invalid
- `"eba"` → 2 distinct chars (`e`, `b`, wait — `e`, `b`, `a` is 3) → invalid, but `"ba"` → 2 distinct → valid, length 2

The longest valid window is `"ece"`, so the answer is **3**.

This is the same family as "at most K distinct characters" — here K is just fixed at 2, so the same sliding window machinery applies, just with a hardcoded threshold instead of a parameter.

## Step 1: Brute Force

Check every substring, and for each one, count its distinct characters using a set. Track the maximum length among substrings where the distinct count is ≤ 2.

```java
public int lengthOfLongestSubstringTwoDistinct(String s) {
    int max = 0;
    for (int i = 0; i < s.length(); i++) {
        Set<Character> seen = new HashSet<>();
        for (int j = i; j < s.length(); j++) {
            seen.add(s.charAt(j));
            if (seen.size() <= 2) {
                max = Math.max(max, j - i + 1);
            } else {
                break;
            }
        }
    }
    return max;
}
```

**Complexity:** O(n²) time in the worst case — for each starting index `i`, we extend `j` until the distinct count exceeds 2, and rebuild the seen-set each time. Space is O(1) since at most 3 characters ever sit in the set before breaking.

## Step 2: Thinking Toward an Optimized Approach

The brute force redoes work it doesn't need to. Every time we move the start of the window forward by one, we throw away the entire distinct-character count we'd already built up and recompute it from scratch. But shrinking a window by one character on the left only affects the count of *that one character* — everything else in the window is untouched.

That observation is the seed of the sliding window optimization: instead of a `Set` that only tells us *whether* a character is present, we want a structure that tells us *how many times* it's present. That way, when we remove a character from the left, we can decrement its count, and only drop it from our "distinct characters" tally when its count hits zero — not the moment we stop seeing it in the current substring, but the moment it's truly gone from the window.

This is why a `HashMap<Character, Integer>` (frequency map) replaces the `HashSet`. The window then grows to the right, expanding the map, and only shrinks from the left when the distinct-character count exceeds 2 — and it shrinks with a `while` loop, not an `if`, because a single removal might not be enough to bring the count back down to a valid state (though for K=2 specifically, since we only ever have 3 distinct chars at the moment of violation, one careful removal usually suffices — the `while` is still the safe, general form).

## My Optimized Solution

```java
HashMap<Character, Integer> freq = new HashMap<>();
    int left = 0, count = 0;
    int max = 0;
    for(int right = 0; right < s.length(); right++)
    {
        if(!freq.containsKey(s.charAt(right)))
            count++;
        freq.put(s.charAt(right), freq.getOrDefault(s.charAt(right), 0) + 1);
        while(count > 2)
        {
            freq.put(s.charAt(left), freq.get(s.charAt(left)) - 1);
            if(freq.get(s.charAt(left)) == 0)
                count--;
            left++;
        }
        max = Math.max(max, right - left + 1);
    }
    return max;
```

### Walking through the logic

1. `freq` is a frequency map tracking how many times each character currently appears inside the window `[left, right]`. `count` tracks how many *distinct* characters are currently in the window — this is the actual thing being constrained, not the map size.
2. For each `right`, we check `!freq.containsKey(s.charAt(right))` **before** inserting — this is what makes `count` accurately reflect "a genuinely new character just entered the window," rather than incrementing on every visit.
3. We then unconditionally `put` the character into `freq`, incrementing its count (or inserting it fresh with `getOrDefault(..., 0) + 1`).
4. The `while(count > 2)` loop is the shrink phase. As long as the window holds more than 2 distinct characters, we shrink from the left:
   - Decrement the frequency of `s.charAt(left)`.
   - If that frequency hits exactly `0`, the character has been fully evicted from the window, so `count--`.
   - Advance `left` regardless — the pointer always moves forward on each shrink iteration, whether or not that particular removal caused an eviction.
5. Once the `while` loop exits, the window `[left, right]` is guaranteed valid (≤ 2 distinct characters), so `max` is updated against the current window size `right - left + 1`.
6. This runs for every `right`, and since `left` only ever moves forward (never resets backward), the two pointers together traverse the string in O(n) total.

### A note on ordering

This solution decrements the frequency **before** checking if it hit zero, which is correct — but it's worth being deliberate about *why* the check comes after the decrement and not before: you're asking "did removing this character just empty it out of the window," which is a post-removal question by definition. Checking before decrementing would answer a different (wrong) question — "is this character about to become empty" — off-by-one against what actually just happened.

## Comparison

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(n²) | O(1) |
| Optimized (Sliding Window + Frequency Map) | O(n) | O(1) — bounded by at most 3 distinct characters in the map at any time |

## A Small Thing Worth Noting

It's tempting to think a `HashSet` would work here since we ultimately only care about *distinct count*, not individual frequencies — and for the brute force, that's true. But the sliding window version specifically needs the frequency map, because shrinking the window requires knowing *when a character's last occurrence has left the window*, not just *whether it's currently present*. A set can tell you presence; it can't tell you "this was the last one." That distinction — count-until-zero versus binary presence — is really the entire reason the data structure upgrades from `Set` to `Map` the moment a window needs to shrink incrementally rather than reset entirely, and it's the same reasoning that will show up again in "at most K distinct characters" with K left general instead of fixed at 2.
