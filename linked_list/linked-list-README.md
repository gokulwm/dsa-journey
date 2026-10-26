# Linked Lists in Java 🔗

> "Understanding the data structure first — its core properties and internal workings — before touching problems."

This section covers the fundamental properties, memory mechanics, and basic operations of Linked Lists in Java.

## 🧠 Core Properties
A linked list is a foundational data structure built from small pieces, called **nodes**, that are chained together by references:

<p align="center">
  <img src="images/Linked_list_properties.png" alt="Linked List Properties" width="80%"><br>
  <em>Figure: Properties of Linked List Data Structure</em>
</p>

*   **Node-Based:** Every node holds two things: the **data** it stores and a **reference** to the next node. The list itself is just a reference to the first node, called the **head**.
*   **Dynamic Length:** There is no fixed size. The list grows by creating a new node and shrinks by dropping one, so no resizing or copying of existing elements is ever needed.
*   **Non-Contiguous Memory:** Nodes can live anywhere on the Heap. The only thing connecting them is the chain of references, which is exactly why the structure is flexible, and exactly why it is slow to index into.
*   **Sequential Access Only:** There is no position arithmetic. To reach the 5th node you must walk through the first four, starting from the head.
*   **Null-Terminated:** The last node's `next` is `null`. This is the signal that the chain has ended, and forgetting to check for it is the most common source of bugs (`NullPointerException`).

**Variants:**
*   **Singly Linked List:** Each node knows only its `next`. Traversal goes one way.
*   **Doubly Linked List:** Each node also holds a `prev` reference, so you can walk both ways and delete a node without needing its predecessor from the head.
*   **Circular Linked List:** The last node points back to the first instead of `null`. There is no natural end, so traversal needs a stopping condition other than `null`.

## ☕ Java-Specific Behaviors
Java has no built-in "node" for you to use directly, so most problems define one themselves:

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}
```

*   **Nodes are Objects:** Every node is an ordinary object living on the Heap. A variable like `head` is just a reference on the Stack pointing at the first node. Each node costs more than the data it holds, because it also carries an object header and a reference to the next node.
*   **`java.util.LinkedList` is a Doubly Linked List:** The class in the Collections framework is doubly linked and implements both `List` and `Deque`. In interviews and DSA problems, however, you almost always work with your own `ListNode` class instead.
*   **Pass by Value (not Reference):** Java is **always pass-by-value**, and that includes node references. A method receives a *copy of the reference*, so it points at the same node as the caller's variable. Changing a node's fields (`node.val = 5`, `node.next = other`) is visible to the caller because both references reach the same object. But *reassigning* the parameter (`node = node.next`) only repoints the local copy; the caller's variable is untouched. This single distinction explains why you cannot "delete the head" by writing `head = head.next` inside a helper method and expect the caller's `head` to change, and why LC 237 can only be solved by overwriting the node's contents instead of replacing the node.
*   **Default Values:** A new node's `next` is `null`, and its data field takes the default of its type (e.g., `0` for `int`).
*   **No Manual Memory Management:** Unlike C, you never `free` a node. Once no reference reaches a node, it becomes unreachable and the garbage collector reclaims it. "Deleting" a node therefore means making it unreachable, which is a pointer operation, not a memory operation.
*   **Aliasing vs. Copying:** Conflating the two is a classic source of bugs.
    *   **Aliasing:** `ListNode b = a;` does **not** copy anything; it creates a second reference to the *same* node. Moving `b` along the list never changes the list, but changing `b.val` or `b.next` changes what `a` sees too.
    *   **Copying a List:** There is no `copyOf` for a chain. A real copy means walking the list and creating a **new node for every element**, so the two lists share no nodes.
    *   **Deep Copy:** When nodes carry extra references (for example a random pointer), a correct copy must also rebuild those relationships among the *new* nodes, not point back into the old list.

## ⚙️ Why Access is O(n)
Arrays get O(1) access from a formula: `Address = Base Address + (Data Type Size × index)`. That formula only works because the elements sit side by side in memory.

A linked list has no such property. Node 3 may be nowhere near node 2 in memory, so there is no address to calculate. The only way to find it is to **follow the chain**: head → node 1 → node 2 → node 3. Reaching index `i` costs `i` hops, so indexing is O(n) in the worst case.

There is a second, quieter cost. Arrays are contiguous, so the CPU loads neighbouring elements into its cache together and traversal runs fast. Linked list nodes are scattered, so each hop can be a cache miss. Two operations with the same Big-O can therefore differ a lot in real running time.

## 🏗️ Basic Operations & Complexity

| Operation | Description | Time Complexity |
| :--- | :--- | :--- |
| **Indexing** | Reaching the i-th node requires walking from the head. | O(n) |
| **Traversal** | Visiting every node. Depends entirely on the length (n). | O(n) |
| **Search (by value)** | Checking node by node until the value is found. | O(n) |
| **Update (by index)** | O(n) to reach the node, then O(1) to change its value. | O(n) |
| **Insert at Head** | Create a node, point it at the old head, move `head`. No shifting. | O(1) |
| **Insert at Tail** | Needs a walk to the last node, unless a `tail` reference is kept. | O(n) / O(1) with tail |
| **Insert after a Known Node** | Two pointer changes once you already hold the node. | O(1) |
| **Insert at Index** | O(n) to find the position, then O(1) to link. | O(n) |
| **Delete Head** | Move `head` forward; the old head becomes garbage. | O(1) |
| **Delete a Node by Value or Index** | You must find the *previous* node to rewire it. | O(n) |
| **Delete a Given Node** | O(1) in a doubly linked list. In a singly linked list it is O(n) to find the predecessor, unless it is not the tail, in which case copying the next node's contents over it gives O(1). | O(1) – O(n) |

*Note: The O(1) cost of insert and delete only applies once you already hold the right node. Linked lists remove the cost of **shifting**, but add the cost of **finding**.*

*Always handle the edge cases: empty list (`head == null`), single node, and changing the head.*

## 🆚 Linked List vs. Array / ArrayList
Both store a sequence, but they make opposite trade-offs:
*   **Memory Layout:** `ArrayList` keeps its elements in one contiguous internal array. A linked list scatters its nodes across the Heap.
*   **Access:** `ArrayList.get(i)` is O(1). `LinkedList.get(i)` is O(n).
*   **Insert/Delete at the Front or Middle:** `ArrayList` has to shift elements (O(n)). A linked list only rewires pointers (O(1)), once the position is already found.
*   **Insert at the End:** `ArrayList` is amortized O(1), with an occasional O(n) resize when the internal array is full. A linked list is O(1) with a tail reference.
*   **Memory Overhead:** Each linked list node stores an extra reference (two for doubly linked), plus object overhead. An array stores just the elements.
*   **In Practice:** `ArrayList` is usually faster even when you insert in the middle, because contiguous memory is so cache-friendly. A linked list earns its place when you already hold node references and need constant-time splicing, or when you must never move or copy elements.

---

## 🧩 Common Problem Patterns

*I will be exploring these in detail with brute-force to optimized solutions in the problem files, but here are the mental models for the core linked list patterns:*

*   **Dummy (Sentinel) Node:** A fake node placed before the real head, so the head never needs special handling. Inserting or deleting at the front looks exactly like inserting or deleting anywhere else.
*   **Slow-Fast Pointers:** Two runners on the same track at different speeds. When the fast one reaches the end, the slow one is at the middle; if the track is a loop, the fast one eventually laps and meets the slow one.
*   **Reversal (Prev / Curr / Next):** Walking along the chain and turning each arrow around as you pass it. The trick is to remember where the next node is *before* you cut the link.
*   **Gap Pointers (Two Pointers, Fixed Distance):** Send one pointer ahead by `k` steps, then move both together. When the leader falls off the end, the follower is exactly `k` from the end.
*   **Merge:** Two sorted queues feeding into one: always take the smaller front node and advance that list.
*   **Overwrite Instead of Rewire:** When you cannot reach the predecessor, change what the node *holds* instead of how it is linked (the idea behind LC 237).
