# Remove Nth Node From End of List - LeetCode 19

## The Problem

You're given the `head` of a singly linked list and an integer `n`. Remove the **n-th node counting from the end** of the list and return the head of the modified list. The constraints guarantee `1 <= n <= size of the list`, so the node always exists.

**Example:** `head = [1, 2, 3, 4, 5]`, `n = 2`

- From the end, the 1st node is `5` and the 2nd node is `4`.
- Remove `4`, and `3` now points straight to `5`.

Output: `[1, 2, 3, 5]`

The tricky part is that a singly linked list only has forward pointers. The "end" is the one place you can't see from the head, so "n-th from the end" can't be read off directly the way an array index can.

---

## Step 1: Brute Force

The most direct approach: walk the list once and store every node in a list, so we can index from the back like an array.

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        List<ListNode> nodes = new ArrayList<>();
        ListNode cur = head;
        while (cur != null) {
            nodes.add(cur);
            cur = cur.next;
        }
        int idx = nodes.size() - n;
        if (idx == 0) return head.next;
        nodes.get(idx - 1).next = nodes.get(idx).next;
        return head;
    }
}
```

**Time: O(L)**, where L is the length of the list. One pass to collect the nodes, then constant work to unlink.
**Space: O(L)**. We hold a reference to every node.

The obvious first fix is that we never needed the nodes themselves, only the *length*. Knowing `L`, the target is node number `L - n` from the front. So count first, then walk:

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        int len = 0;
        ListNode cur = head;
        while (cur != null) {
            len++;
            cur = cur.next;
        }
        int idx = len - n;
        if (idx == 0) return head.next;
        cur = head;
        for (int i = 1; i < idx; i++) {
            cur = cur.next;
        }
        cur.next = cur.next.next;
        return head;
    }
}
```

**Time: O(L)**, **Space: O(1)**. This is the "Better" tier: two passes, no extra memory. The cost is that we spend an entire traversal just to learn where the end is.

---

## Step 2: Thinking Toward an Optimized Approach

Both versions above are really answering one question first: *where is the end?* The brute force answers it by remembering everything, the better version by counting everything. Either way a full traversal is spent before any real work begins, and the root cause is structural. A singly linked list gives you the **start for free** (`head`) but the **end only by walking**. Pointers only point forward, so there's no cheap way to approach the end "from the other side."

That asymmetry suggests a different move: instead of fighting it, **change which end is the start**. If the list were flipped, then "n-th from the end" would become "n-th from the beginning", and the beginning is exactly the thing we can reach instantly. Counting from the front needs no length and no extra memory, just `n - 1` steps forward.

Flipping a linked list doesn't need extra space either. Reversal can be done in place by rewiring each node's `next` to point backwards while carrying three pointers, one for the node behind, one for the current node, and one for the node ahead so the rest of the list isn't lost when the link is cut. That's O(L) time and O(1) space, which is the same budget as the better version above. So the plan is to **reverse the list, delete the n-th node from the front, and reverse again** so the caller gets the original orientation back.

Two small details fall out of this reasoning. First, deleting a node in a singly linked list requires a handle on the node *before* it, so the walk needs a trailing pointer that stays one step behind. Second, the head of the reversed list has no predecessor at all, so removing it (the case `n == 1`, i.e. the original tail) can't be done by rewiring `prev.next` and has to be handled on its own.

---

## My Optimized Solution

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        head = reverseList(head);
        if(n == 1)
        return reverseList(head.next);
        ListNode temp = head;
        ListNode prev = head;
        for(int i = 1;i < n;i++)
        {
            prev = temp;
            temp = temp.next;
        }
        prev.next = temp.next;
        return reverseList(head);
    }
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode cur = head;
        ListNode nxt = head;
        while(cur != null)
        {
            nxt = nxt.next;
            cur.next = prev;
            prev = cur;
            cur = nxt;
        }
        return prev;
    }
}
```

### Walking through the logic

1. **`reverseList`** is the standard three-pointer reversal. **`nxt = nxt.next`** advances first so the rest of the list survives, **`cur.next = prev`** flips the link, then **`prev = cur`** and **`cur = nxt`** step everything forward. When `cur` falls off the end, `prev` is the new head.
2. **`head = reverseList(head)`** flips the list, so the original tail is now at the front and "n-th from the end" has become "n-th from the start".
3. **`if(n == 1) return reverseList(head.next)`** handles the case where the node to delete is the new head. There's no predecessor to rewire, so we skip it by reversing from `head.next` onward, which also restores the original order. If the list had only one node, `head.next` is `null` and `reverseList(null)` returns `null`, which is the correct empty result.
4. **`temp`** and **`prev`** both start at the head of the reversed list. `temp` will land on the node to delete and `prev` will trail one step behind it.
5. **`for(int i = 1; i < n; i++)`** runs `n - 1` times. Each iteration does **`prev = temp`** then **`temp = temp.next`**, so afterwards `temp` sits on the n-th node and `prev` on the (n-1)-th.
6. **`prev.next = temp.next`** unlinks the target by making its predecessor skip over it.
7. **`return reverseList(head)`** flips the list back, so the result is in the original order with the n-th-from-end node gone.

**Trace on `head = [1, 2, 3, 4, 5]`, `n = 2`:**

After the first reversal the list is `5 → 4 → 3 → 2 → 1`.

| step | prev | temp | list state |
|------|------|------|------------|
| start | 5 | 5 | 5 → 4 → 3 → 2 → 1 |
| i = 1 | 5 | 4 | 5 → 4 → 3 → 2 → 1 |
| `prev.next = temp.next` | 5 | 4 | 5 → 3 → 2 → 1 |
| final reverse | – | – | 1 → 2 → 3 → 5 |

Result: `[1, 2, 3, 5]` ✅

And for `n = 1` on the same list: reversed is `5 → 4 → 3 → 2 → 1`, the `n == 1` branch returns `reverseList(4 → 3 → 2 → 1)` which is `1 → 2 → 3 → 4` ✅

> **Note (no change made to the code):** The solution relies on the constraint `1 <= n <= size`. If `n` exceeded the list length, `temp` would run off the end and throw a `NullPointerException`, but LeetCode guarantees that can't happen. `nxt = head` in `reverseList` is only a placeholder initial value since it's reassigned before use. The input list is temporarily mutated (reversed) and then restored, which is fine here but would matter if other code held references to the original nodes during the call. Finally, the braceless `if(n == 1)` with its `return` on the next line works correctly, though adding a line under it later would silently change the meaning.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (store all nodes in a list) | O(L) | O(L) | Simple, but holds every node |
| Better (count length, then walk L − n) | O(L) | O(1) | Two passes, no extra memory |
| **Reverse, delete, reverse (mine)** | **O(L)** | **O(1)** | About three passes and temporarily mutates the list; same big-O as Better |
| Fast/slow pointers with dummy node | O(L) | O(1) | Single pass, no mutation; the standard optimal answer |

Honest reading of this table: all four are O(L) time, so the real differences are in extra memory and the number of passes. Your solution matches the best space bound but does more traversals than the two-pass version, and the fast/slow pointer method beats both on passes. It's a valid and clever solution, just not the leanest.

---

## A Small Thing Worth Noting

The one-pass solution that sits below your row in spirit is built on a **fixed gap invariant**. Start two pointers at a dummy node placed before the head, advance the first pointer `n` steps ahead, then move both together until the first reaches the last node. Because the gap between them stays exactly `n`, the second pointer ends up *right before* the node to delete, with no length and no reversal needed. The dummy node also removes the special case for deleting the head, which is the same problem your `n == 1` branch solves.

The two ideas are worth keeping side by side because they attack the same obstacle (the end of a singly linked list is unreachable without a walk) from opposite directions. Reversal **changes the coordinate system** so the end becomes the start. The gap trick **keeps the coordinate system** but lets one pointer discover the end on behalf of the other. A good recognition rule: **"k-th from the end" in a singly linked list → either measure the length, reverse, or maintain a fixed gap between two pointers.** The same family shows up in finding the middle of a list (LC 876, fast moves twice as fast), deleting the middle node (LC 2095), rotating a list by `k` (LC 61), and checking a palindromic list (LC 234, where reversing the second half is exactly this trick).
