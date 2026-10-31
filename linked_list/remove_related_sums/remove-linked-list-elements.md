# Remove Linked List Elements - LeetCode 203

## The Problem

You're given the `head` of a singly linked list and an integer `val`. Remove **every** node whose value equals `val` and return the new head. The nodes that remain must keep their original relative order.

**Example:** `head = [1, 2, 6, 3, 4, 5, 6]`, `val = 6`

- The `6` at index 2 is removed ✅
- The `6` at the tail is removed ✅
- Everything else stays in place.

Output: `[1, 2, 3, 4, 5]`

The tricky part is that the node you want to delete might be the **head itself** (or the first several nodes), and a linked list gives you no way to "reach back" and fix the previous node's pointer once you've moved on.

---

## Step 1: Brute Force

The most direct translation: forget about pointer surgery, copy the values you want to keep into an array, then build a fresh list from that array.

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        ArrayList<Integer> keep = new ArrayList<>();
        ListNode temp = head;
        while (temp != null) {
            if (temp.val != val) keep.add(temp.val);
            temp = temp.next;
        }
        ListNode dummy = new ListNode(-1);
        ListNode tail = dummy;
        for (int x : keep) {
            tail.next = new ListNode(x);
            tail = tail.next;
        }
        return dummy.next;
    }
}
```

**Time: O(n)**. One pass to collect, one pass to rebuild.
**Space: O(n)**. The array plus a brand-new copy of every kept node.

The obvious first fix is that we never needed new nodes or a side array. The nodes we want to keep already exist, so we can relink them in place. The usual way to do that is to handle the "head might be removed" case separately, then walk with a `prev` pointer:

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        while (head != null && head.val == val) {
            head = head.next;
        }
        ListNode curr = head;
        while (curr != null && curr.next != null) {
            if (curr.next.val == val) {
                curr.next = curr.next.next;
            } else {
                curr = curr.next;
            }
        }
        return head;
    }
}
```

**Time: O(n)**, **Space: O(1)**. This is the "Better" tier: optimal complexity, but it needs a special-case loop just for the head.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force is already linear in time, so the waste here isn't repeated work, it's **memory and copying**. Allocating a whole new list to avoid touching pointers is overkill, because every node we want to keep is already sitting in the original list. The real job is just to decide, node by node, "does this one survive?" and to wire the survivors together.

The awkward part of in-place deletion is the head. Every other node has a predecessor whose `next` we can rewrite, but the head has none, so the head case needs its own loop. The way around this is to **give the head a predecessor**: place a throwaway **dummy node** in front of the list. Now every real node, including the original head, has something pointing at it, and the head stops being special.

From there, think of it as building the answer rather than deleting from the input. Keep one pointer that sweeps the original list, and a second pointer that always marks the **tail of the answer built so far**. When the sweeping node is a keeper, attach it to the answer's tail and advance the tail. When it's a node to remove, simply don't attach it. Nothing is ever "deleted"; removed nodes just never get linked in.

One detail falls out of this reasoning. Kept nodes still carry their **old `next` pointers**, which may lead straight into nodes we meant to drop. If the last kept node is followed by removed nodes in the original list, it will still point at them. So after the sweep finishes, the tail of the answer must be explicitly terminated.

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
    public ListNode removeElements(ListNode head, int val) {
        ListNode temp = head;
        ListNode re = new ListNode(-1);
        ListNode ret = re;
        while(temp != null)
        {
            if(temp.val != val)
            {
                ret.next = temp;
                ret = ret.next;
            }
            temp = temp.next;
        }
        ret.next = null;
        return re.next;
    }
} 
```

### Walking through the logic

1. **`temp`** is the sweeping pointer that walks the original list from `head`. **`re`** is a dummy node holding `-1`, and **`ret`** starts at `re` and always points to the **tail of the answer built so far**.
2. **`new ListNode(-1)`** gives the answer a fixed starting node, so the original head needs no special treatment. The dummy's value is never read, so `-1` is arbitrary.
3. **`if(temp.val != val)`** decides whether the current node is a keeper.
4. **`ret.next = temp; ret = ret.next;`** attaches the keeper to the end of the answer and advances the tail onto it.
5. **`temp = temp.next`** runs on every iteration, kept or not. A node to remove is skipped simply by never being attached.
6. **`ret.next = null`** terminates the answer. The last kept node may still point into removed nodes from the original list, and this line cuts that stale link.
7. **`return re.next`** skips the dummy and returns the real head. If every node was removed, `re.next` is `null`, which is the correct empty list.

**Trace on `head = [1, 2, 6, 3, 6]`, `val = 6`:**

| temp.val | keep? | action | answer built so far (from dummy) |
|----------|-------|--------|----------------------------------|
| 1 | yes | attach, `ret` → 1 | -1 → 1 |
| 2 | yes | attach, `ret` → 2 | -1 → 1 → 2 |
| 6 | no | skip | -1 → 1 → 2 |
| 3 | yes | attach, `ret` → 3 | -1 → 1 → 2 → 3 → 6 (stale link) |
| 6 | no | skip | -1 → 1 → 2 → 3 → 6 (stale link) |
| end | – | `ret.next = null` cuts `3 → 6` | -1 → 1 → 2 → 3 |

Result: `[1, 2, 3]` ✅

> **Note (no change made to the code):** The `ret.next = null` line is essential, not cosmetic. Without it, the trace above would return `[1, 2, 3, 6]`, because node `3` still points at the trailing `6`. Also, `temp = temp.next` reads the old pointer *after* `ret.next = temp` has run, which is safe here because `ret` is always at or behind `temp`, so attaching never overwrites a pointer you haven't read yet.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (array + rebuild list) | O(n) | O(n) | Copies every kept value into new nodes |
| Better (in place, special-case head) | O(n) | O(1) | Needs a separate loop for leading matches |
| **Dummy node + tail pointer (mine)** | **O(n)** | **O(1)** | Same complexity as Better; no head special case |

Mine ties the Better approach on complexity. The gain is not speed but **simplicity**: one loop, no head case.

---

## A Small Thing Worth Noting

The dummy node is one of the most reusable tricks in linked list problems. Whenever the **head might change** (removal, insertion before the head, merging, partitioning), a dummy in front turns the head into an ordinary node and removes a whole class of edge cases. The same pattern shows up in Merge Two Sorted Lists (LC 21), Partition List (LC 86), and Remove Duplicates from Sorted List II (LC 82).

The other idea worth remembering is the **"build the answer with a tail pointer"** style: rather than deleting from the input, you decide which nodes to link into a new chain. Always ask what stale `next` pointers the last attached node might still carry, because that is exactly the bug `ret.next = null` prevents. A recursive version (`head.next = removeElements(head.next, val)`) is also valid and elegant, but costs O(n) stack space, so it isn't better than this one.
