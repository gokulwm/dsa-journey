# Two Pointers (Array Patterns): Summary

Two pointers is an essential technique used to optimize brute-force nested loops (`O(n^2)`) into linear `O(n)` (or `O(n log n)` if sorting is required) scans. By coordinating two indices across a data structure, you can process elements in a single pass.

This family splits into two main sub-groups based on how the pointers move:

1. **Same Direction** — Both pointers move forward. Used for sliding windows (tracking a contiguous segment) or read-write operations (in-place array modification).
2. **Opposite Direction** — Pointers start at opposite ends and converge. Used heavily for sorted arrays, symmetric checks, and boundary shrinking.

```text
Two Pointers
├── Same Direction
│   ├── Sliding Window
│   │   ├── Fixed size          → window length is a given constant k
│   │   └── Variable size
│   │       ├── Maximum/Longest → expand greedily, shrink only when invalid
│   │       └── Minimum/Smallest→ expand until valid, shrink aggressively while valid
│   └── Read–Write (non-window) → in-place compaction, filtering, deduplication
└── Opposite Direction
    └── Converging              → Start at ends, move inward (sorted arrays, palindromes)
```

## Pattern Comparison Table

| Pattern | Window size | Expand / Move Left Rule | Shrink / Move Right Rule | Typical use case |
|---|---|---|---|---|
| **Fixed Sliding Window** | Constant `k` | Add next element | Remove element leaving the window every slide | Per-window sum/average/frequency check |
| **Variable Window — Maximum** | Grows/shrinks | Expand every step | Shrink **only when invalid** | Longest substring/subarray under a constraint |
| **Variable Window — Minimum** | Grows/shrinks | Expand until valid | Shrink **aggressively while still valid** | Smallest window satisfying a condition |
| **Read–Write (Fast–Slow)** | N/A — no window | N/A | N/A | In-place removal/compaction, order-preserving filtering |
| **Opposite Direction (Converging)**| N/A | `left++` (increases sum/skips invalid) | `right--` (decreases sum/skips invalid) | Pair sums in sorted arrays, Palindromes, Max Area |

## Visual Cheat Sheet

```text
Fixed Window:      [l────r]                (size constant)
                       [l────r]

Variable (Max):    [l──r]                  expand normally
                    [l────────r]           keep expanding while valid
                       [l─────r]           shrink only when broken

Variable (Min):    [l────────r]            first valid window (wide)
                       [l─────r]           shrink while still valid
                             [l r] invalid  stop, go back to expanding

Read-Write:        w
                    r → → → →              read scans everything (w advances on "keep")

Converging:        l →            ← r      move inward based on sorted/symmetric rules
```

## Decision Guide

Ask these in order:

1. **Is a window size explicitly given/fixed (`k`)?**
   → **Fixed sliding window.**
2. **Does the problem ask for the longest/maximum contiguous segment under a constraint** (e.g., "at most K distinct", "no repeats")?
   → **Variable sliding window — maximum.** Expand greedily, shrink only on violation.
3. **Does the problem ask for the minimum/smallest contiguous segment satisfying a condition** (e.g., "smallest subarray with sum ≥ target", "minimum window containing all chars")?
   → **Variable sliding window — minimum.** Expand to first validity, then shrink aggressively.
4. **Does the problem say "in-place", "remove", "compact", "partition", "move to end/front" while preserving order?**
   → **Read–write two pointers.** No window notion — just a lagging write boundary.
5. **Does the problem ask for pair sums in a sorted array, or symmetric checks like palindromes?**
   → **Opposite direction (Converging).** Move left/right pointers inward based on value comparisons.

## Why This Family Is `O(n)`

In every one of these patterns, each element is:
- Processed, added, or read **at most once**, and
- Removed or bypassed **at most once**.

Since pointers never move backward (or only move inward towards each other), the total number of pointer movements across the whole run is bounded by `n` (or `2n`), giving `O(n)` time overall. Note: Converging pointer problems might require an initial `O(n log n)` sort.

## Common Pitfalls Across the Whole Family

- **Off-by-one errors** on window length — it's always `r - l + 1`, and the element leaving a fixed window is always at `r - k`.
- **Sliding window with negative numbers** for sum-based conditions — the "expand grows the sum, shrink shrinks it" assumption breaks down.
- **Confusing "shrink until valid" with "shrink while valid"** — mixing up the maximum-window and minimum-window shrink disciplines.
- **Applying converging pointers to unsorted arrays** without sorting them first.
- **Forgetting to guard against edge cases** — empty arrays, `k` larger than the array length, or "no valid window exists" cases.

## Files in This Series

| File | Pattern |
|---|---|
| `01-fixed-sliding-window.md` | Fixed-size sliding window |
| `02-variable-sliding-window-maximum.md` | Variable-size window — longest/maximum |
| `03-variable-sliding-window-minimum.md` | Variable-size window — smallest/minimum |
| `04-read-write-two-pointers.md` | Read–write (fast–slow) in-place compaction |
| `opposite-direction-two-pointers.md` | Opposite direction (converging) array scanning |
