# Delete Nodes From Linked List Present in Array - LeetCode 3217

## The Problem

You're given an integer array `nums` and the `head` of a singly linked list. Remove every node whose value appears in `nums`, and return the head of the modified list. The values in `nums` are unique, and the list is guaranteed to keep at least one node.

**Example:** `nums = [1, 3]`, `head = [1, 2, 3, 4, 5]`

- `1` is in `nums`, so that node is removed ❌
- `2` is not in `nums`, so it stays ✅
- `3` is in `nums`, so that node is removed ❌
- `4` and `5` are not in `nums`, so they stay ✅

Output: `[2, 4, 5]`

The tricky part is twofold: checking "is this value in `nums`?" quickly for every node, and unlinking nodes safely, especially when the node being removed is the **head**, which has no predecessor.

---

## Step 1: Brute Force

Walk the list, and for each node scan the whole `nums` array to see whether its value is present. A dummy node in front of the head removes the special case for deleting the head.

```java
class Solution {
    public ListNode modifiedList(int[] nums, ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode prev = dummy;
        ListNode curr = head;
        while (curr != null) {
            boolean found = false;
            for (int x : nums) {
                if (x == curr.val) {
                    found = true;
                    break;
                }
            }
            if (found) {
                prev.next = curr.next;
            } else {
                prev = curr;
            }
            curr = curr.next;
        }
        return dummy.next;
    }
}
```

**Time: O(L · n)**, where `L` is the number of list nodes and `n = nums.length`. Every node may trigger a full scan of `nums`. With both up to 10⁵, that is on the order of 10¹⁰ operations in the worst case.
**Space: O(1)**. Only a few pointers.

---

## Step 2: Thinking Toward an Optimized Approach

The linked list traversal itself is already as cheap as it can be: every node has to be looked at at least once, so O(L) is the floor for that part. The waste is entirely in the membership test. For each node, the brute force asks "is this value in `nums`?" by searching an unsorted array from scratch, and it asks that same kind of question again for every node, never reusing anything it learned.

So the real question is how to make "is this value in `nums`?" cost O(1) instead of O(n). That is exactly what a **hash set** is for. Pay once, up front, to load all of `nums` into a set in O(n), and every later membership check becomes a constant-time lookup. The expensive repeated search collapses into one preprocessing pass plus cheap queries, and the total drops from a product, `L · n`, to a sum, `L + n`.

The second part of the reasoning is about pointers. To delete a node from a singly linked list, you must change the `next` of the node *before* it, so the traversal needs to remember a `prev` pointer that trails `temp`. The key detail is that `prev` should only advance when a node is **kept**. When a node is deleted, `prev` stays where it is, because the next node to examine will also need to hang off the same `prev`. The head is the one awkward case, since it has no predecessor at all: deleting it means reassigning `head` itself, and that has to be repeated if the new head is also in the set.

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
    public ListNode modifiedList(int[] nums, ListNode head) {
        HashSet<Integer> set = new HashSet<>();
        for(int i = 0;i < nums.length;i++)
            set.add(nums[i]);
        ListNode temp = head;
        ListNode prev = null;
        while(temp != null)
        {
            if(set.contains(temp.val))
            {
                if(temp == head)
                {
                    ListNode newHead = temp.next;
                    temp.next = null;
                    head = newHead;
                    temp = head;
                    continue;
                }
                else
                {
                    prev.next = temp.next;
                }
            }
            else
                prev = temp;
            temp = temp.next;
        }
        return head;
    }
}
```

### Walking through the logic

1. **`set`** holds every value from `nums`, so each later membership check is O(1) on average. Building it costs O(n).
2. **`temp`** is the node currently being examined, and **`prev`** is the last node that was *kept*. `prev` starts as `null` because nothing has been kept yet.
3. **`set.contains(temp.val)`** decides whether the current node must be deleted.
4. **`if(temp == head)`** handles the special case of deleting the front of the list. There is no `prev` to relink, so instead the head moves forward: **`head = newHead`**, and **`temp = head`** restarts the examination at the new front.
5. **`temp.next = null`** detaches the removed head from the rest of the list. This isn't required for correctness, but it cleanly severs the removed node.
6. **`continue`** skips the bottom `temp = temp.next`, because `temp` was already set to the new head. Moving on would skip a node that might also need deleting. Consecutive removed heads are handled by repeating this branch.
7. **`prev.next = temp.next`** is the normal deletion for a non-head node: the kept node before it now points past it. Note that `prev` is *not* updated here, since the node after the deleted one still hangs off the same `prev`.
8. **`else prev = temp`** advances `prev` only when the current node is kept.
9. **`temp = temp.next`** moves the traversal forward.

**Trace on `nums = [1, 3]`, `head = [1, 2, 3, 4, 5]`:**

| step | temp.val | in set? | action | prev | head after | list after |
|------|----------|---------|--------|------|------------|------------|
| 1 | 1 | yes | `temp == head`, so head moves to 2 | null | 2 | 2→3→4→5 |
| 2 | 2 | no | keep, `prev = 2` | 2 | 2 | 2→3→4→5 |
| 3 | 3 | yes | `prev.next = temp.next` (2→4) | 2 | 2 | 2→4→5 |
| 4 | 4 | no | keep, `prev = 4` | 4 | 2 | 2→4→5 |
| 5 | 5 | no | keep, `prev = 5` | 5 | 2 | 2→4→5 |

Result: `[2, 4, 5]` ✅

> **Note (no change made to the code):** `HashSet` needs `import java.util.*;`, which LeetCode adds automatically but a local file would not. The head-special-case logic is correct: `prev` is `null` exactly while no node has been kept, which is the same moment `temp == head`, so the `else` branch never dereferences a null `prev`. One possible simplification is a dummy node before the head, which removes the head branch entirely (as in the brute force above), but the current approach works as written.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (scan `nums` per node) | O(L · n) | O(1) | Up to ~10¹⁰ operations, far too slow at the limits |
| **HashSet + single pass (mine)** | **O(L + n)** | **O(n)** | Asymptotically optimal; trades memory for time |
| Boolean array indexed by value | O(L + n + M) | O(M) | Same big-O class; constant-factor win when values are bounded by `M` (here 10⁵) |

---

## A Small Thing Worth Noting

The pattern here is two ideas stacked together: **"replace a repeated search with a one-time preprocessing structure"** and **"to delete from a singly linked list, you need the node before it."** The first is the same trade that turns O(n²) into O(n) in Two Sum and in prefix-sum counting. The second is the source of the classic head problem: the head is the only node with no predecessor, so it forces either a special case (as in this solution) or a **dummy node** placed before the head, which makes every node look like a middle node. Whenever a linked list problem might delete the head, reach for a dummy node first.

Also note that because the constraints bound values to 1..10⁵, a plain `boolean[]` can replace the hash set. It avoids boxing and hashing overhead, which is why it often runs faster in practice even though the big-O is the same.

Related variations: Remove Linked List Elements (LC 203, same idea with a single target value), Remove Duplicates from Sorted List II (LC 82, where whole runs of nodes are deleted), and Delete Node in a Linked List (LC 237, where you only have access to the node itself).
