# Delete Node in a Linked List - LeetCode 237

## The Problem

You're handed a single node from the middle of a singly linked list, but **not the head**. Your job is to remove that node from the list. The catch is that you can't reach the node *before* it, so the usual `prev.next = node.next` rewiring is off the table. You only see `node` and everything after it. The problem guarantees `node` is never the tail.

**Example:** list `4 → 5 → 1 → 9`, `node` = the node holding `5`

- Before: `4 → 5 → 1 → 9`
- After: `4 → 1 → 9` ✅

Output: the list reads `4 → 1 → 9` when traversed from the head.

The tricky part is not removing a node, it's removing one when you have no access to whatever points at it.

---

## Step 1: Brute Force

Since we can't unlink the node directly, treat everything from `node` onward as a small array: collect the values after `node`, write them back starting at `node`, and cut off the now-redundant last node.

```java
class Solution {
    public void deleteNode(ListNode node) {
        List<Integer> vals = new ArrayList<>();
        ListNode curr = node.next;
        while (curr != null) {
            vals.add(curr.val);
            curr = curr.next;
        }

        curr = node;
        int i = 0;
        while (i < vals.size()) {
            curr.val = vals.get(i++);
            if (i == vals.size()) {
                curr.next = null;
            } else {
                curr = curr.next;
            }
        }
    }
}
```

**Time: O(n)**. One pass to collect the values after `node`, one pass to write them back, where n is the number of nodes from `node` to the tail.
**Space: O(n)**. The `vals` list holds a copy of every value after `node`.

This works, but the extra list is pure overhead: we copy values out only to copy them straight back in.

---

## Step 2: Thinking Toward an Optimized Approach

Look at what the brute force actually does to the list: every node from `node` onward ends up holding the value of the node right after it. The auxiliary list is only a courier. Each value moves exactly one step to the left, so at the moment we overwrite `curr.val`, the value we need is already sitting in `curr.next.val`. We can read it straight from the neighbor and drop the list entirely. That removes the O(n) space and leaves a single in-place left-shift of values along the chain, ending with the tail being detached.

Pushing that thought one step further is where the real insight is. If every node ends up holding its successor's value, then the shift only truly **needs to happen once**. After `node` takes `node.next`'s value, `node.next` is a duplicate of what `node` already holds, and everything after it is already correct. So rather than continuing to shift the rest of the chain, we can skip the duplicate by pointing `node` past it. The whole deletion collapses to "become your next node, then bypass your next node", with no traversal at all.

---

## My Optimized Solution

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public void deleteNode(ListNode node) {
        ListNode temp = node;
        while(temp != null)
        {
            if(temp.next.next == null)
            {
                temp.val = temp.next.val;
                temp.next = null;
            }
            else
                temp.val = temp.next.val;
            temp = temp.next;
        }
    }
}
```

### Walking through the logic

1. **`temp = node`** starts the walk at the node we were given. The loop runs while `temp != null`.
2. **`temp.next.next == null`** asks: is the next node the tail? This is how the code spots the last step *before* walking off the end. Since the problem guarantees `node` is not the tail, `temp.next` is never `null` when the loop starts, so this check is always safe.
3. **`else` branch: `temp.val = temp.next.val`** copies the successor's value into the current node, shifting one value left.
4. **`if` branch: `temp.val = temp.next.val; temp.next = null`** does the same copy for the last pair, then severs the old tail so the list is one node shorter.
5. **`temp = temp.next`** moves forward. After the `if` branch, `temp.next` was just set to `null`, so `temp` becomes `null` and the `while` condition ends the loop. No counter or flag is needed.

**Trace on `4 → 5 → 1 → 9`, `node` = `5`:**

| Iteration | temp at | temp.next.next == null? | Branch | List after |
|-----------|---------|--------------------------|--------|------------|
| 1 | `5` | no (it's `9`) | `else`: `5` takes `1` | `4 → 1 → 1 → 9` |
| 2 | `1` (second one) | yes | `if`: takes `9`, cut tail | `4 → 1 → 9` |
| – | `null` | – | loop ends | `4 → 1 → 9` |

Result: `4 → 1 → 9` ✅

> **Note (no change made to the code):** this solution is correct and passes, but it isn't the tightest possible. It shifts values along the *entire* remainder of the list, which is O(n) time, while the same effect needs only one copy and one pointer change (`node.val = node.next.val; node.next = node.next.next;`). Also, the inner `if/else` repeats `temp.val = temp.next.val` in both branches, which is harmless but could be written once.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (collect values, rewrite) | O(n) | O(n) | Extra list used only to ferry values one step left |
| **Shift values along the chain (mine)** | **O(n)** | **O(1)** | Drops the extra list, but still walks to the tail |
| Optimal (copy next, bypass next) | O(1) | O(1) | `node.val = node.next.val; node.next = node.next.next;` |

My solution beats brute force on space but not on time, so the optimal approach sits below it.

---

## A Small Thing Worth Noting

This problem doesn't actually delete the node you were given. It *impersonates* deletion: the given node absorbs its successor's identity (its value and its `next` pointer), and it's the **successor's node object** that effectively disappears. That's invisible when you traverse from the head, which is why it passes, but it would break if anything else held a reference to the successor node, because that reference would now point at a node no longer in the list.

The same mindset, **change what a node holds rather than how it's linked**, applies whenever you can't touch the predecessor. It's also why the tail is excluded: the tail has no successor to copy from, so the trick has nothing to work with. A good recognition rule: **"remove a node" + "no access to the previous node" → overwrite state from the next node instead of rewiring.** The same idea shows up whenever a structure forbids direct access to a parent or predecessor and you have to make the node itself carry the change.
