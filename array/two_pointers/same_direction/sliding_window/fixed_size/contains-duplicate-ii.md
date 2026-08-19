# Contains Duplicate II - LeetCode  219

## The Problem 

Given an integer array `nums` and an integer `k`, return `true` if there are two **distinct** indices `i` and `j` in the array such that `nums[i] == nums[j]` and `abs(i - j) <= k`.

**Example:**
```
Input: nums = [1,2,3,1], k = 3
Output: true
Explanation: nums[0] = 1 and nums[3] = 1, and abs(0 - 3) = 3 <= k
```

```
Input: nums = [1,0,1,1], k = 1
Output: true
Explanation: nums[2] = 1 and nums[3] = 1, and abs(2 - 3) = 1 <= k
```

The key thing to notice: this isn't just "does a duplicate exist" — it's "does a duplicate exist **close enough** to another copy of itself". A duplicate that's far apart (index distance more than `k`) doesn't count.

---

## Step 1: Brute Force

The most direct reading of the problem: for every index `i`, look at every other index `j` within distance `k`, and check if the values match.

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j <= i + k && j < nums.length; j++) {
                if (nums[i] == nums[j]) return true;
            }
        }
        return false;
    }
}
```

For each `i`, we check up to `k` elements ahead of it. That gives us roughly `n * k` comparisons in the worst case.

**Complexity:**
- Time: O(n · k)
- Space: O(1)

This works, but it's doing a lot of repeated scanning. If `k` is large, this gets slow fast. We're re-checking overlapping ranges over and over without remembering anything from the previous iteration.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force is wasteful because it forgets everything between iterations. Every time `i` moves forward, we throw away all the comparison work we just did and start scanning again from scratch.

What we actually need is a way to answer one question fast: **"have I seen this value recently — within the last `k` indices?"**

That's a lookup problem, not a scanning problem. A hash-based structure is the natural fit here, since it gives O(1) lookups.

### Better: HashMap storing last seen index

One straightforward way — walk through the array once, and for every value, remember the **last index** where you saw it. When you see the value again, just check if the current index minus the last seen index is within `k`.

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        Map<Integer, Integer> lastSeen = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (lastSeen.containsKey(nums[i]) && i - lastSeen.get(nums[i]) <= k) {
                return true;
            }
            lastSeen.put(nums[i], i);
        }
        return false;
    }
}
```

**Complexity:**
- Time: O(n) — one pass, O(1) map operations
- Space: O(n) — in the worst case (all unique values), the map holds every element

This is a huge improvement in time. But space-wise, it's not great — the map can grow to hold the entire array, even though we only ever care about the last `k` elements at any point. Anything older than `k` indices back is dead weight sitting in the map.

---

## My Optimized Solution

Since we only ever care about a window of the last `k+1` elements (current index plus `k` positions back), why keep anything older than that around at all? Instead of a map that grows forever, use a set that represents exactly a sliding window of size `k+1` — old elements get evicted the moment they fall outside the window.

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] arr, int k) {
        Set<Integer> set = new HashSet<>();
        int l = 0, r = k+1;
        if(arr.length <= k)
        {
            for(int i = 0;i < arr.length;i++)
            {
                if(set.contains(arr[i])) return true;
                else set.add(arr[i]);
            }
            return false;
        }
        for(int i = 0;i <= k;i++)
        {
            if(set.contains(arr[i])) return true;
            else set.add(arr[i]);
        }
        while(r < arr.length)
        {
            set.remove(arr[l]);
            if(set.contains(arr[r])) return true;
            else set.add(arr[r]);
            l++;
            r++;
        }
        return false;
    }
}
```

**Walking through it:**

1. **Edge case first:** if `arr.length <= k`, then no matter where two equal elements sit in the array, their index distance can never exceed `k` (the array isn't even that long). So this collapses to a plain "does any duplicate exist anywhere" check — one pass, no window needed.

2. **Build the first window:** for the general case, load up a set with the first `k+1` elements (indices `0` through `k`). While doing this, if the same value shows up twice already inside this window, we're done — return `true` immediately.

3. **Slide the window:** now move the window one step at a time. `l` tracks the element leaving the window, `r` tracks the element entering it.
   - Remove `arr[l]` from the set — it's now outside the window, so it shouldn't be able to trigger a false match anymore.
   - Check if `arr[r]` (the new element entering) is already in the set. If yes, we found two equal values within `k` of each other — return `true`.
   - Otherwise add `arr[r]` to the set and move both pointers forward.

4. If the window slides all the way to the end without a hit, return `false`.

**Complexity:**
- Time: O(n) — each element is added to and removed from the set at most once
- Space: O(min(n, k)) — the set never holds more than `k+1` elements at a time

---

## Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n · k) | O(1) | Rescans overlapping ranges repeatedly, no memory between iterations |
| Better (HashMap, last seen index) | O(n) | O(n) | Fast lookups, but map can grow to hold the whole array |
| Optimal (Sliding Window Set) | O(n) | O(min(n, k)) | Only ever remembers exactly the elements inside the current window |

---

## A Small Thing Worth Noting

The HashMap approach and the sliding window approach are both O(n) in time — so on paper they look equally fast. The real difference shows up in **space**. The HashMap keeps a permanent record of every value's last position, even for values that are long out of range and can never contribute to an answer anymore. The sliding window version actively evicts anything that falls outside the `k`-distance, so its memory footprint is bounded by `k` instead of `n`.

This is a nice example of a pattern worth remembering: whenever a problem only cares about a *fixed distance* or *fixed range* between elements, a sliding window often beats a plain hashmap — not because it's faster in time, but because it only remembers exactly what it needs to, and nothing more.
