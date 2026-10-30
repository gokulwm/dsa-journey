# Delete the Middle Node of a Linked List - LeetCode 2095

## The Problem

You're given the `head` of a singly linked list. Delete the **middle node** and return the head of the modified list. For a list of `n` nodes, the middle node is the one at 0-based index `⌊n / 2⌋`. That means for an even-length list, it is the *second* of the two central nodes.

**Example:** `head = [1, 3, 4, 7, 1, 2, 6]`

- `n = 7`, so the middle index is `⌊7 / 2⌋ = 3`
- The node at index 3 holds `7`, so it is removed ❌
- Every other node stays ✅

Output: `[1, 3, 4, 1, 2, 6]`

Two more cases to keep in mind: `[1, 2, 3, 4]` becomes `[1, 2, 4]` (index 2 is the middle), and `[2, 1]` becomes `[2]`. A single-node list `[1]` becomes empty (`null`).

The tricky part is that a singly linked list can't tell you its length or its middle for free, and deleting a node requires reaching the node *before* it.

---

## Step 1: Brute Force

The most direct translation: copy every node reference into a list so that indexing is possible, then relink around index `n / 2`.

```java
class Solution {
    public ListNode deleteMiddle(ListNode head) {
        List<ListNode> nodes = new ArrayList<>();
        for (ListNode t = head; t != null; t = t.next) {
            nodes.add(t);
        }
        int n = nodes.size();
        if (n == 1) return null;
        int mid = n / 2;
        nodes.get(mid - 1).next = nodes.get(mid).next;
        return head;
    }
}
```

**Time: O(n)**. One pass to collect the nodes, then constant work to relink.
**Space: O(n)**. The list holds a reference to every node, which is wasteful when we only need two of them.

The obvious first fix is that we don't need to store anything. We only need the length `n`, and then we can walk straight to the node *before* the middle:

```java
class Solution {
    public ListNode deleteMiddle(ListNode head) {
        int n = 0;
        for (ListNode t = head; t != null; t = t.next) n++;
        if (n == 1) return null;
        ListNode curr = head;
        for (int i = 0; i < n / 2 - 1; i++) {
            curr = curr.next;
        }
        curr.next = curr.next.next;
        return head;
    }
}
```

**Time: O(n)**, **Space: O(1)**. This is the "Better" tier. The memory problem is gone, but the list is traversed twice: about `n` steps to count, then about `n / 2` more to reach the target.

---

## Step 2: Thinking Toward an Optimized Approach

Both versions above hinge on knowing `n`, or on being able to index into the list. But `n` is only needed for one reason: to find where the halfway point is. The list never hands us that directly, so we pay a full traversal just to learn a number, and then walk again to act on it.

The way out is to find the middle **without ever knowing the length**. Think of two runners on the same track: if one runs at double the speed of the other, then at the moment the fast runner reaches the end, the slow runner has covered exactly half the distance. That means the slow runner is standing at the middle. This is the **fast and slow pointer** idea: `slow` moves one node per step, `fast` moves two, and when `fast` runs off the end, `slow` is at the middle. The whole job collapses into a single pass.

There is one more thing the reasoning has to account for. Deleting a node from a singly linked list means changing the `next` of the node *before* it, but `slow` lands *on* the middle, not before it. So we need to remember where `slow` was one step ago, which means a trailing `prev` pointer that is updated right before `slow` advances. The remaining details are the tiny lists: a one-node list has no node left after deleting its only (middle) node, so it must return `null`, and it is worth checking how the loop behaves for two and three nodes so that `prev` is always valid when we use it.

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
    public ListNode deleteMiddle(ListNode head) {
        if(head.next == null)
        return null;
        if(head.next.next == null)
        {
            head.next = null;
            return head;
        }
        ListNode slow = head;
        ListNode fast = head;
        ListNode prev = slow;
        while(fast != null && fast.next != null)
        {
            prev = slow;
            slow = slow.next;
            fast = fast.next.next;
        }
        prev.next = slow.next;
        slow.next = null;
        return head;
    }
}
```

### Walking through the logic

1. **`if(head.next == null) return null`** handles the single-node list. Its only node is the middle, so deleting it leaves an empty list.
2. **`if(head.next.next == null)`** handles the two-node list. The middle is index 1 (the second node), so **`head.next = null`** removes it and the head is returned unchanged.
3. **`slow`** and **`fast`** both start at `head`. **`prev`** starts there too and will trail one step behind `slow`.
4. **`while(fast != null && fast.next != null)`** keeps going as long as `fast` can still take a full double step. Checking `fast` first and `fast.next` second avoids a null dereference.
5. **`prev = slow`** is executed *before* `slow` moves, so `prev` always sits exactly one node behind `slow`.
6. **`slow = slow.next`** and **`fast = fast.next.next`** advance the two runners at their different speeds.
7. When the loop exits, **`slow`** is on the middle node (index `⌊n / 2⌋`). For an even `n`, `fast` falls off the end as `null`; for an odd `n`, `fast` stops on the last node.
8. **`prev.next = slow.next`** bypasses the middle node, and **`slow.next = null`** detaches the removed node cleanly.

**Trace on `head = [1, 3, 4, 7, 1, 2, 6]` (indices 0 to 6):**

| iteration | prev | slow | fast | continue? |
|-----------|------|------|------|-----------|
| start | idx 0 (1) | idx 0 (1) | idx 0 (1) | yes |
| 1 | idx 0 (1) | idx 1 (3) | idx 2 (4) | yes |
| 2 | idx 1 (3) | idx 2 (4) | idx 4 (1) | yes |
| 3 | idx 2 (4) | idx 3 (7) | idx 6 (6) | no (`fast.next` is null) |

At the exit, `slow` is the node `7` at index 3, and `prev` is the node `4`. Then `prev.next = slow.next` makes `4 → 1`, giving:

Result: `[1, 3, 4, 1, 2, 6]` ✅

> **Note (no change made to the code):** the two-node special case (`head.next.next == null`) is actually redundant. For `[a, b]`, the loop runs once: `prev = a`, `slow = b`, `fast = null`, and then `prev.next = slow.next` sets `a.next = null`, which is the same result. Only the single-node guard is truly necessary. Also, `prev = slow` at its declaration is only an initial placeholder, since the loop always overwrites it before use for lists of three or more nodes. Neither point is a bug, so the code is correct as written.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (store all nodes in a list) | O(n) | O(n) | Direct indexing, but wastes memory on every node |
| Better (count length, then walk) | O(n) | O(1) | Constant space, but about 1.5 traversals |
| **Fast/slow pointers (mine)** | **O(n)** | **O(1)** | Single pass, constant space; the standard optimal answer |

All three share the same big-O in time. The improvements are in memory (row 1 to 2) and in the number of traversals (row 2 to 3), so the difference between the last two is a constant factor, not an asymptotic one.

---

## A Small Thing Worth Noting

The loop condition decides *which* middle you land on for even-length lists. With `fast != null && fast.next != null`, `slow` ends at the **second** of the two central nodes (index `n / 2`), which is exactly the node this problem wants deleted. If you tightened the condition to `fast.next != null && fast.next.next != null`, `slow` would stop one node earlier, at the **first** central node (index `n / 2 - 1`). That is the useful variant when you need the node *before* the middle, and it would let you drop the separate `prev` pointer entirely.

Recognition rule: **"find the middle / halfway point of a linked list in one pass" → fast and slow pointers.** The same technique powers cycle detection (a faster runner eventually laps a slower one), finding the start of a cycle, and splitting a list in half for merge sort or for checking palindromes.

Related variations: Middle of the Linked List (LC 876, the same pointer pattern without the deletion), Remove Nth Node From End of List (LC 19, two pointers with a fixed gap instead of a fixed speed ratio), Linked List Cycle (LC 141), and Palindrome Linked List (LC 234).
