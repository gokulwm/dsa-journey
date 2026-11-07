# Remove Duplicates from Sorted List - LeetCode 83

## The Problem

You're given the `head` of a **sorted** singly linked list. Delete duplicates so that each value appears **exactly once**, and return the resulting list (still sorted).

**Example:** `head = [1, 1, 2, 3, 3]`

- `1` appears twice, so keep the first and drop the second ✅
- `2` appears once, so it stays ✅
- `3` appears twice, so keep the first and drop the second ✅

Output: `[1, 2, 3]`

The tricky part is that, unlike LC 82, one copy of a duplicated value **survives**. So the first node of every group is always kept, which means the head can never be deleted. The work is entirely about rewiring `next` pointers so the dropped nodes are skipped.

---

## Step 1: Brute Force

The most direct translation: for every node, scan everything after it and unlink any later node that has the same value. This doesn't even use the fact that the list is sorted.

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        for (ListNode a = head; a != null; a = a.next) {
            ListNode b = a;
            while (b.next != null) {
                if (b.next.val == a.val) b.next = b.next.next;
                else b = b.next;
            }
        }
        return head;
    }
}
```

**Time: O(n²)**. For each of the n nodes we may walk the whole remainder of the list.
**Space: O(1)**. Only a few pointers, all edits are in place.

The obvious first fix is to stop rescanning. A single pass can remember which values have already been seen in a `HashSet<Integer>`, and unlink any node whose value is already in the set:

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) return head;
        HashSet<Integer> seen = new HashSet<>();
        seen.add(head.val);
        ListNode cur = head;
        while (cur.next != null) {
            if (seen.contains(cur.next.val)) {
                cur.next = cur.next.next;
            } else {
                seen.add(cur.next.val);
                cur = cur.next;
            }
        }
        return head;
    }
}
```

**Time: O(n)**, **Space: O(n)**. This is the "Better" tier, and it would work even on an unsorted list, but that generality is exactly what we're paying for with the set, when the problem has promised us sorted input.

---

## Step 2: Thinking Toward an Optimized Approach

Both earlier versions answer "have I seen this value before?" with rescanning or with extra memory. But sorted order changes what that question means. In a sorted list, all copies of a value sit in one contiguous run, so a node is a duplicate **if and only if it equals the node just before it among the survivors**. There is nothing to remember beyond the last node we decided to keep.

That suggests a different mental model: instead of deleting nodes, **build the answer by choosing which nodes to link together**. Keep one pointer on the last node accepted into the result, and a second scanner that walks forward through every node. Each time the scanner finds a node whose value differs from the last accepted one, that node starts a new value group, so it gets attached after the last accepted node and becomes the new "last accepted." Nodes that equal the last accepted value are simply never attached, so they fall out of the chain without any explicit deletion step.

Two small details fall out of this reasoning. First, the head is always the first node of its group, so it is always accepted and no special handling of the head is needed. Second, because we only rewire a `next` pointer at the moment we accept a new node, the **last accepted node's `next` is never updated for the trailing duplicates**. If the list ends with a run of equal values, the last accepted node still points into that run, so the chain has to be explicitly terminated once the scan finishes.

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
        ListNode prev = head;
        ListNode nxt = head.next;
        while(nxt != null)
        {
            if(prev.val != nxt.val)
            {
                prev.next = nxt;
                prev = prev.next;
            }
            nxt = nxt.next;
        }
        prev.next = null;
        return head;
    }
}
```

### Walking through the logic

1. **`if(head == null || head.next == null) return head;`** handles the trivial cases: an empty list or a single node cannot contain duplicates.
2. **`prev`** is the last node accepted into the result, which starts as **`head`** since the head is always kept. **`nxt`** is the scanner, starting at **`head.next`**.
3. **`prev.val != nxt.val`**: the scanner has reached a node with a new value. Because the list is sorted, this means a new group has begun.
4. **`prev.next = nxt`** attaches that node right after the last accepted one, bypassing any duplicates in between, and **`prev = prev.next`** makes it the new last accepted node.
5. **`nxt = nxt.next`** advances the scanner every iteration, whether or not the node was accepted. Equal-valued nodes are skipped just by moving past them.
6. **`prev.next = null`** after the loop cuts the tail. If the list ended with duplicates, `prev` still points at them, and this line severs that link.

**Trace on `head = [1, 1, 2, 3, 3]`** (subscripts distinguish equal values):

| step | prev | nxt | prev.val != nxt.val? | action | chain from head |
|------|------|-----|----------------------|--------|-----------------|
| start | 1ₐ | 1ᵦ | – | – | 1ₐ→1ᵦ→2→3ₐ→3ᵦ |
| 1 | 1ₐ | 1ᵦ | no | skip, `nxt` moves on | 1ₐ→1ᵦ→2→3ₐ→3ᵦ |
| 2 | 1ₐ | 2 | yes | `1ₐ.next = 2`, `prev = 2` | 1ₐ→2→3ₐ→3ᵦ |
| 3 | 2 | 3ₐ | yes | `2.next = 3ₐ`, `prev = 3ₐ` | 1ₐ→2→3ₐ→3ᵦ |
| 4 | 3ₐ | 3ᵦ | no | skip, `nxt` becomes null | 1ₐ→2→3ₐ→3ᵦ |
| end | 3ₐ | null | – | `prev.next = null` | 1ₐ→2→3ₐ |

Result: `[1, 2, 3]` ✅

> **Note (no change made to the code):** The logic is correct, and the final `prev.next = null` is essential: on `[1, 1]` or the trace above, without it the tail duplicates would remain reachable. Observations for your reference. (1) The `head.next == null` half of the early return is redundant, since a single node makes the loop run zero times and `prev.next = null` is then harmless, though `head == null` is still needed to avoid dereferencing `head.next`. (2) `prev = prev.next` is the same as `prev = nxt` at that moment, since you just set `prev.next = nxt`. (3) The skipped nodes are never individually unlinked; they are simply left out of the chain, which is fine because nothing references them afterward. (4) The list is modified in place, and the nodes that were skipped become unreachable garbage.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (rescan after every node) | O(n²) | O(1) | In place, works on unsorted input too |
| Better (HashSet of seen values) | O(n) | O(n) | Fast and general, but ignores sortedness |
| **Two pointers, stitch survivors (mine)** | **O(n)** | **O(1)** | One pass, uses sorted adjacency, no extra memory |

His solution already matches the best known complexity. The common alternative, a single pointer that does `cur.next = cur.next.next` whenever it sees an equal neighbor, has the same O(n) time and O(1) space; the difference is only style (unlinking duplicates one at a time versus stitching survivors together).

---

## A Small Thing Worth Noting

Compare this with LC 82 right beside it in your repo. There, a duplicated value has to vanish **entirely**, so the first node of a group can be deleted, the head can change, and you needed a flag and a dummy-node idea to cope. Here, one copy always survives, so the first node of every group is automatically safe, the head never moves, and a plain pair of pointers is enough. A good recognition rule: **"keep one copy" means the group's first node is anchored, "remove all copies" means it isn't, and the second case is where the head-handling headaches (and the dummy node) come from.**

Your trailing `prev.next = null` also deserves to be remembered as a pattern: whenever you build a result by **linking selected nodes rather than deleting the others**, the last selected node still carries its old `next`, and you must terminate the chain yourself. The same idea recurs in problems like partitioning a list (LC 86) and odd-even list rearrangement (LC 328). The array counterparts of this problem are Remove Duplicates from Sorted Array (LC 26) and its "at most twice" variant (LC 80).
