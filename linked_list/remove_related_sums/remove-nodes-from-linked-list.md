# Remove Nodes From Linked List - LeetCode 2487

## The Problem

You're given the `head` of a singly linked list. Remove every node that has a node with a **strictly greater** value somewhere to its right, and return the head of the modified list. Nodes with equal values do not remove each other.

**Example:** `head = [5, 2, 13, 3, 8]`

- `5` has `13` to its right, so it is removed ❌
- `2` has `13` to its right, so it is removed ❌
- `13` has nothing greater to its right, so it stays ✅
- `3` has `8` to its right, so it is removed ❌
- `8` is the last node, so it stays ✅

Output: `[13, 8]`

The tricky part is that a node's fate depends on everything *after* it, but a singly linked list only lets you walk forward.

---

## Step 1: Brute Force

The most direct translation: for every node, scan everything to its right. If any later node is bigger, drop it, otherwise keep it.

```java
class Solution {
    public ListNode removeNodes(ListNode head) {
        ListNode dummy = new ListNode(-1);
        ListNode tail = dummy;
        for (ListNode cur = head; cur != null; cur = cur.next) {
            boolean keep = true;
            for (ListNode r = cur.next; r != null; r = r.next) {
                if (r.val > cur.val) {
                    keep = false;
                    break;
                }
            }
            if (keep) {
                tail.next = cur;
                tail = cur;
            }
        }
        return dummy.next;
    }
}
```

**Time: O(n²)**. Each node can trigger a scan of the entire remainder of the list, for example on a strictly decreasing list where every node is kept and every scan runs to the end.
**Space: O(1)**. Only a few pointers; the kept nodes are relinked in place.

---

## Step 2: Thinking Toward an Optimized Approach

The brute force keeps asking the same question from scratch: "is anything to my right bigger than me?" But the answers for neighbouring nodes are heavily related. If node `a` is smaller than the next node `b`, then `a` is doomed, and we learned that without looking any further. The scan for `a` and the scan for `b` are mostly re-reading the same suffix.

So flip the direction of the reasoning. Instead of each node looking right, let each **new node judge the nodes before it**. Walk left to right, and keep a collection of nodes that are still *candidates for survival*. When a new node arrives with a larger value, every candidate smaller than it has just found its "greater node to the right," so they can all be thrown out immediately. Candidates that are bigger than or equal to the newcomer are still safe for now.

That collection has a very specific shape. Anything smaller than the newcomer is removed from the back, and the newcomer is added at the back, so we only ever touch **one end**. That is exactly a **stack**, and because we only keep nodes that haven't been beaten, the values in it are always **non-increasing from bottom to top**. This is the classic **monotonic stack**. Each node is pushed once and popped at most once, so the total work across the whole walk is linear, even though a single step may pop many nodes.

Two small details fall out of this. First, the comparison must be **strictly greater** to pop, because equal values do not remove each other. Second, whatever is left in the stack at the end is precisely the answer, in the original left-to-right order, so all that remains is relinking those surviving nodes.

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
    public ListNode removeNodes(ListNode head) {
      ListNode temp = head;
      Stack<ListNode> st = new Stack<>();
      st.push(temp);
      temp = temp.next;
      while(temp != null)
      {
        if(temp.val > st.peek().val)
        {
            while(!st.empty() && temp.val > st.peek().val )
            {
                st.pop();
            }
        }
        st.push(temp);
        temp = temp.next;
      }  
      ListNode ret = new ListNode(-1);
      temp = ret;
      for(ListNode n : st)
      {
        temp.next = n;
        temp = temp.next;
      }
      temp.next = null;
      return ret.next;
    }
}
```

### Walking through the logic

1. **`st`** is a stack of `ListNode`s holding the nodes that are still candidates to survive. **`temp`** is the walking pointer.
2. **`st.push(temp)`** seeds the stack with the head. The first node has nothing before it to judge, so it always starts as a candidate. Then **`temp = temp.next`** moves on to the second node.
3. **`temp.val > st.peek().val`** checks whether the current node is bigger than the node on top of the stack. If it is, at least the top node has just found a greater node to its right.
4. **`while(!st.empty() && temp.val > st.peek().val)`** pops every candidate that is strictly smaller than the current node. Because the stack is non-increasing from bottom to top, the nodes that must go are always sitting at the top, and the popping stops as soon as we hit a node that is `>=` the current one.
5. **`st.push(temp)`** then adds the current node as a candidate. This is always done, since a node can only be beaten by something that comes *after* it.
6. After the walk, **`ret`** is a dummy head. **`for(ListNode n : st)`** iterates the surviving nodes and **`temp.next = n`** stitches them together. A Java `Stack` is backed by `Vector`, so the for-each loop goes from **bottom to top**, which is the original left-to-right order.
7. **`temp.next = null`** terminates the new list, and **`return ret.next`** skips the dummy.

**Trace on `head = [5, 2, 13, 3, 8]`:**

| step | temp.val | stack before | action | stack after (bottom → top) |
|------|----------|--------------|--------|----------------------------|
| seed | 5 | [] | push head | [5] |
| 1 | 2 | [5] | 2 > 5? no, push | [5, 2] |
| 2 | 13 | [5, 2] | 13 > 2 pop, 13 > 5 pop, push | [13] |
| 3 | 3 | [13] | 3 > 13? no, push | [13, 3] |
| 4 | 8 | [13, 3] | 8 > 3 pop, 8 > 13? no, push | [13, 8] |

Relink the stack in bottom-to-top order: `13 -> 8 -> null`

Result: `[13, 8]` ✅

> **Note (no change made to the code):** A few small things worth knowing. (1) The outer `if(temp.val > st.peek().val)` is redundant, since the `while` condition already performs the same check, so the `while` alone would do the job. (2) `temp.next = null` is harmless but also redundant here: the last node of the original list is always the last one pushed, so it is always the top of the stack and its `next` is already `null`. (3) `Stack` and `ListNode` need `import java.util.*;` locally, which LeetCode adds automatically. (4) `st.push(temp)` at the start assumes `head` is non-null, which the constraints guarantee (at least one node). (5) The code reuses the original nodes rather than allocating new ones, so the only extra memory is the stack itself.

---

## Comparison Table

| Approach | Time Complexity | Space Complexity | Notes |
|----------|-----------------|------------------|-------|
| Brute Force (scan right for every node) | O(n²) | O(1) | Re-reads the same suffix repeatedly |
| **Monotonic Stack (mine)** | **O(n)** | **O(n)** | Each node pushed and popped at most once |
| Reverse, track running max, reverse back | O(n) | O(1) | Same time, no stack; mutates the list twice |

Honest note: the stack solution is optimal in time, but it is not the best in space. Reversing the list turns "greater to my right" into "greater than the max so far", which needs only a single variable, so it gets the same O(n) time with O(1) extra space.

---

## A Small Thing Worth Noting

The recognition rule here is: **"remove or keep elements based on whether something greater (or smaller) exists to the right" → monotonic stack.** The linked list is almost incidental. The same idea works on arrays, and the only real difference is that on a list you can't index backwards, so the stack replaces the random access you'd otherwise use.

Notice also that the direction of the question can be flipped. "Does anything to my right beat me?" is the same as "when something arrives, who does it beat?" Whenever one direction forces a rescan, try the other one. This is also why the reverse-and-track-max solution works: reading the list right-to-left makes the "greater to the right" condition a simple running maximum.

Related variations: Next Greater Element I (LC 496), Daily Temperatures (LC 739), Next Greater Node In Linked List (LC 1019), Remove K Digits (LC 402), and Largest Rectangle in Histogram (LC 84).
