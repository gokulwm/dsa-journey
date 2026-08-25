# Repeated DNA Sequences - LeetCode 187

## The Problem

The DNA sequence is made up of the letters `A`, `C`, `G`, `T`. Given a string `s` representing a DNA sequence, return all 10-letter-long substrings that occur more than once in `s`.

**Example:**
`s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"`

Sliding a window of size 10 across the string:
- Window at index 0: `"AAAAACCCCC"` → first time seen, count = 1
- Window at index 10: `"AAAAACCCCC"` → seen again! count = 2 → this is a repeat
- Window at index 1: `"AAAACCCCCA"` → count = 1 (unique)
- ...and so on for every window of size 10.

Any substring whose count ends up greater than 1 goes into the result. Here, the answer is `["AAAAACCCCC", "CCCCCAAAAA"]`.

---

## Step 1: Brute Force

The direct idea is: take every window of size 10, and compare it against every other window of size 10 to check for a match.

```java
class Solution {
    public List<String> findRepeatedDnaSequences(String s) {
        List<String> result = new ArrayList<>();
        Set<String> seen = new HashSet<>();
        Set<String> added = new HashSet<>();

        for (int i = 0; i + 10 <= s.length(); i++) {
            String window = s.substring(i, i + 10);
            if (seen.contains(window) && !added.contains(window)) {
                result.add(window);
                added.add(window);
            } else {
                seen.add(window);
            }
        }
        return result;
    }
}
```

**Complexity:**
- Time: O(n · L) where `n` is the number of windows and `L = 10` is the window length (extracting each substring costs O(L), and hashing/comparing it costs O(L) too).
- Space: O(n · L) for storing the seen substrings.

*(Note: even this version already uses a set to avoid a full O(n²) pairwise comparison — a truly naive brute force, comparing every window against every other window directly, would be O(n² · L). Either way, the point is the same: we're not yet counting occurrences in a single clean pass.)*

---

## Step 2: Thinking Toward an Optimized Approach

Instead of just tracking whether we've *seen* a window before, we can directly count how many times each 10-letter window appears as we slide across the string once. A `HashMap<String, Integer>` lets us do exactly that: each window becomes a key, and we bump its count every time it reoccurs.

Once we've swept through the entire string, any key in the map with a count greater than 1 is, by definition, a repeated sequence. This turns the problem into a single linear pass over the string, sliding the window one character at a time and updating the map — the same fixed sliding window shape used in the earlier problems, just with a map counting substrings instead of characters.

---

## My Optimized Solution

```java
class Solution {
    public List<String> findRepeatedDnaSequences(String s) {
        if(s.length() < 10)
        return new ArrayList<String>();
        HashMap<String, Integer> combo = new HashMap<>();
        combo.put(s.substring(0, 10), 1);
        int left = 1;
        int right = 11;
        String sam;
        while(right <= s.length())
        {
            sam = s.substring(left++, right++);
            combo.put(sam, combo.getOrDefault(sam, 0) + 1);
        }
        List<String> res = new ArrayList<>();
        for(Map.Entry<String, Integer> entry : combo.entrySet())
        {
            if(entry.getValue() > 1)
                res.add(entry.getKey());
        }
        return res;
    }
}
```

### Walking through the logic

1. **Handle the edge case**: if the string is shorter than 10 characters, no valid window even exists, so return an empty list immediately.
2. **Seed the map with the first window**: take the substring from index 0 to 10 and put it in `combo` with a count of 1.
3. **Set up sliding pointers**: `left` starts at 1, `right` starts at 11 — this represents the *next* window (shifted one step from the first).
4. **Slide across the string**: while `right` hasn't gone past the end of the string, extract the substring from `left` to `right`, then use `getOrDefault` to either start its count at 1 or increment it if it's already in the map. Both pointers move forward by one each iteration.
5. **Collect the repeats**: after the full sweep, loop through every entry in the map. Any substring with a count greater than 1 gets added to the result list.
6. **Return the result**: the final list contains every 10-letter sequence that appeared more than once.

---

## Comparison

| Approach | Time | Space |
|---|---|---|
| Brute Force (Set-based) | O(n · L) | O(n · L) |
| Optimized (HashMap counting) | O(n · L) | O(n · L) |

*(Both approaches share the same asymptotic complexity here since window length `L = 10` is fixed and small — the real win of the optimized version is that it counts occurrences directly in one clean pass, rather than juggling two sets to detect and avoid duplicate additions.)*

---

## A Small Thing Worth Noting

This solution builds actual `String` objects for every window using `substring()`, which under the hood copies 10 characters each time. Since DNA only has 4 possible letters (`A`, `C`, `G`, `T`), each one can be encoded in just 2 bits, meaning a full 10-letter window fits into a single 20-bit integer. A more advanced version of this problem uses a rolling integer hash built this way instead of extracting substrings at all — avoiding string allocation entirely and making each step of the slide a cheap bitwise operation. It doesn't change the overall time complexity here (since `L = 10` is constant either way), but it's a good technique to know for when window content needs to be compared very frequently at scale.
