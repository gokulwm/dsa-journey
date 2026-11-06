# Remove Duplicates from Sorted List II - LeetCode 82

## The Problem

You're given the `head` of a **sorted** linked list. Delete every node whose value appears more than once, so that only values that were originally distinct remain, and return the resulting list (still sorted).

**Example:** `head = [1, 2, 3, 3, 4, 4, 5]`

- `1` appears once, so it stays ✅
- `2` appears once, so it stays ✅
- `3` appears twice, so **both** `3`s go ❌
- `4` appears twice, so **both** `4`s go ❌
- `5` appears once, so it stays ✅

Output: `[1, 2, 5]`

The tricky part is that this is not "remove extra copies" (that's LC 83, where one `3` survives). Here a value with any duplicate disappears entirely, which also means the **head itself can be deleted**, e.g. `[1, 1, 1, 2, 3]` becomes `[2, 3]`.

---

## Step 1: Brute Force

The most direct translation: for every node, scan the whole list to count how many nodes share its value. If the count is exactly 1, append that node to a result list built behind a dummy node.

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        for (ListNode i = head; i != null; i = i.next) {
            int count = 0;
            for (ListNode j = head; j != null; j = j.next) {
                if (j.val == i.val) count++;
            }
            if (count == 1) {
                tail.next = i;
                tail = tail.next;
            }
        }
        tail.next = null;
        return dummy.next;
    }
}
```

**Time: O(n²)**. For each of the n nodes we re-scan up to n nodes to count its value.
**Space: O(1)**. Only pointers and a counter (the result reuses the existing nodes).

The obvious first fix is that re-counting is pure waste. Count every value **once** in a frequency map, then make a second pass that keeps only nodes whose count is 1:

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        HashMap<Integer, Integer> freq = new HashMap<>();
        for (ListNode n = head; n != null; n = n.next) {
            freq.put(n.val, freq.getOrDefault(n.val, 0) + 1);
        }
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        for (ListNode n = head; n != null; n = n.next) {
            if (freq.get(n.val) == 1) {
                tail.next = n;
                tail = n;
            }
        }
        tail.next = null;
        return dummy.next;
    }
}
```

**Time: O(n)**, **Space: O(n)**. This is the "Better" tier. It's linear, but it pays for a hash map and completely ignores the one thing the problem hands us for free: the list is sorted.

---

## Step 2: Thinking Toward an Optimized Approach

Both versions above answer "how many times does this value occur?" as if the list were in random order. But the list is sorted, and sortedness changes the question entirely. In a sorted list, equal values are **contiguous**: every value forms a single run, and a node is a duplicate exactly when it matches an immediate neighbor. There is nothing to count globally. A single left-to-right sweep, looking only at `cur` and `cur.next`, is enough to discover every run and decide its fate on the spot. That removes the hash map and drops the space to O(1).

What remains is pure pointer bookkeeping, and that's where the real difficulty of this problem lives. We need to keep a handle on the **last confirmed survivor** (a `prev` pointer), because when a whole run is deleted we must reconnect that survivor to whatever comes after the run. Deleting a run means relinking `prev.next` past it, not just dropping nodes one at a time.

There's also a subtle detection problem. When `cur` reaches the **last** node of a run, its next neighbor already differs, so a plain comparison says "not a duplicate" even though it belongs to a run that must be deleted. So the algorithm has to *remember* that it was inside a run, which means carrying one bit of state (a flag) across iterations. One more consequence: since the head can be part of a deleted run, there has to be a way to move the head itself (either a special case or a dummy node in front of it), and whatever run is still "open" when the loop ends needs to be handled after the loop too.

---

## My Optimized Solution

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null || head.next == null)
        return head;
        boolean setflag = false;
        ListNode prev = head;
        ListNode cur = head;
        while(cur.next != null)
        {
            if(cur.val == cur.next.val)
            {
                cur.next = cur.next.next;
                setflag = true;
            }
            else
            {
                if(setflag)
                {
                    if(cur == head)
                    {
                        head = cur.next;
                        prev = head;
                    }
                    else
                        prev.next = cur.next;
                    cur= cur.next;
                    setflag = false;
                }
                else
                {
                    prev = cur;
                    cur = cur.next;
                }
            }
        }
        if(setflag)
        {
            if(cur == head)
            {
                head = cur.next;
                prev = head;
            }
            else
                prev.next = cur.next;
            cur= cur.next;
            setflag = false;
        }
        return head;
    }
}
```

### Walking through the logic

1. **`if(head == null || head.next == null) return head;`** handles the trivial cases: an empty list or a single node can't contain duplicates.
2. **`prev`** always points to the last node *known to survive*, and **`cur`** is the node being examined. **`setflag`** records "the node `cur` sits on is part of a duplicate run that still has to be deleted."
3. **`cur.val == cur.next.val`** means `cur` has a duplicate right behind it. **`cur.next = cur.next.next`** splices out that following copy, and **`setflag = true`** records that `cur` itself is now doomed. `cur` does not move, so a run of any length keeps collapsing into the single node `cur`.
4. In the `else` branch (`cur.val != cur.next.val`) the run, if any, has ended. If **`setflag`** is true, `cur` is the lone leftover copy of a duplicated value and must go too.
5. **`if(cur == head)`** covers the case where the deleted node is the head itself: **`head = cur.next`** moves the head forward, and **`prev = head`** keeps `prev` consistent with the new front. Otherwise **`prev.next = cur.next`** unlinks `cur` by bridging the last survivor to the node after it.
6. **`cur = cur.next; setflag = false;`** moves on and resets the state for the next run. Notice that `prev` is *not* advanced after a deletion, because the node it points to is still the last survivor.
7. If **`setflag`** is false, `cur` is unique so far: **`prev = cur; cur = cur.next;`** confirms it as a survivor and advances both pointers.
8. The loop exits when `cur.next == null`, i.e. `cur` is the final node. If the list **ends inside a run** (like `[..., 4, 4]`), `setflag` is still true, because the "run ended" branch never got a chance to fire. The block after the loop repeats the same deletion logic once to clean that up.

**Trace on `head = [1, 2, 3, 3, 4, 4, 5]`:**

| Step | cur | cur.next | Branch taken | List after step | prev | setflag |
|------|-----|----------|--------------|-----------------|------|---------|
| 1 | 1 | 2 | differ, flag off → keep | 1→2→3→3→4→4→5 | 1 | false |
| 2 | 2 | 3 | differ, flag off → keep | 1→2→3→3→4→4→5 | 2 | false |
| 3 | 3 | 3 | equal → splice next | 1→2→3→4→4→5 | 2 | true |
| 4 | 3 | 4 | differ, flag on → delete `cur` via `prev.next` | 1→2→4→4→5 | 2 | false |
| 5 | 4 | 4 | equal → splice next | 1→2→4→5 | 2 | true |
| 6 | 4 | 5 | differ, flag on → delete `cur` via `prev.next` | 1→2→5 | 2 | false |

Loop ends (`cur` is `5`, `cur.next == null`), `setflag` is false, so nothing left to clean up.

Result: `[1, 2, 5]` ✅

> **Note (no change made to the code):** the solution is correct on all the cases I traced (head deleted, tail run deleted, whole list duplicates → `null`, runs of length 3+). A few observations only. (1) The cleanup block after the loop duplicates the in-loop deletion logic; its trailing assignments `cur = cur.next`, `prev = head` and `setflag = false` are never read afterwards, so they're dead stores. (2) The `cur == head` special case exists only because the head can be deleted; a dummy node placed before `head` removes that branch and the duplicated block entirely. (3) Splicing nodes out leaves them unreachable, which Java's garbage collector handles, so no manual cleanup is needed.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (re-count each value) | O(n²) | O(1) | Rescans the whole list per node |
| Better (HashMap frequency + rebuild) | O(n) | O(n) | Two passes; ignores that the list is sorted |
| Dummy node + skip runs | O(n) | O(1) | Same complexity as mine, no head special case |
| **Pointer pass with flag (mine)** | **O(n)** | **O(1)** | One pass, in place; handles head via `cur == head` |

Asymptotically my solution is as good as it gets, since every node must be looked at at least once. The dummy-node version isn't faster, just structurally simpler, so I've listed it as a peer rather than an improvement.

---

## A Small Thing Worth Noting

The core recognition rule here is: **"sorted list + duplicates" means duplicates are adjacent, so use neighbor comparison instead of counting.** That single property is what moves the problem from O(n) space to O(1). The same observation powers sorted-array dedup problems, which makes it worth checking for sortedness before reaching for a hash map.

The second, more transferable lesson is about **linked-list edits that can change the head**. Whenever the first node may be deleted, the head becomes a special case that infects the code (as the `cur == head` branches show). The standard cure is a **dummy node** sitting before the head, so every real node, including the first, has a `prev`. The flag in this solution is also doing something specific: it carries "I was inside a run" information across the moment when the run ends, which a plain `val != next.val` comparison cannot see. Closely related variations: LC 83 (remove duplicates but keep one copy, no flag needed), LC 26/80 (the array versions of dedup), and LC 203 (remove elements by value, where the dummy node idea first becomes essential).
