# Maximum Average Subarray I - LeetCode 643

## The Problem - LeetCode 643

We are given an array of integers `nums` and a number `k`. We need to find `k` consecutive elements (a subarray of length `k`) whose average is the biggest among all such subarrays, and return that average.

**Example:**
```
nums = [1, 12, -5, -6, 50, 3], k = 4
```
If we check every group of 4 consecutive numbers:
- `[1, 12, -5, -6]` → sum = 2, avg = 0.5
- `[12, -5, -6, 50]` → sum = 51, avg = 12.75
- `[-5, -6, 50, 3]` → sum = 42, avg = 10.5

The best one is `[12, -5, -6, 50]`, so the answer is `12.75`.

## Step 1: Brute Force

My first instinct, like most people's, was the direct way: try every possible window of size `k`, add up its elements from scratch, and check the average.

```java
class Solution {
    public double findMaxAverage(int[] nums, int k) {
        double ans = Double.NEGATIVE_INFINITY; //For cases of maximum findings
        double cur;
        for (int i = 0; i <= nums.length - k; i++) {
            cur = 0.0;
            for (int j = i; j < i + k; j++)
                cur += nums[j];
            cur = cur / k;
            ans = Math.max(ans, cur);
        }
        return ans;
    }
}
```

**How it works:** the outer loop `i` picks the starting point of the window, and the inner loop `j` walks through all `k` elements from `i` to `i + k - 1` and adds them up. Once I have the sum, I divide by `k` to get the average, and keep the biggest one seen so far in `ans`.


**Complexity:**
- Time: `O(n * k)` — for each of the `n - k + 1` windows, we do `k` work to sum it.
- Space: `O(1)`

This works, but it repeats a lot of unnecessary work. Every time the window slides by one, most of its elements are the same as the previous window — yet the brute force sums them all over again.

## Step 2: Thinking Toward an Optimized Approach

The key thing here is that the subarray length `k` is **fixed**. It never changes size, it only moves.

So the question becomes: why recalculate the sum of each window from scratch? If I already know the sum of one window, moving to the next window only changes two things:
- One element leaves from the left side
- One new element joins from the right side

So instead of re-adding all `k` elements every time, I can just do:
```
new_sum = old_sum - (element leaving) + (element entering)
```

This is basically a **sliding window** — the window "slides" one step to the right each time, and I update the sum in O(1) instead of recalculating it in O(k).

## My Optimized Solution

```java
class Solution {
    public double findMaxAverage(int[] nums, int k) {
        double prev, curr;
        double sum = 0;
        int l = 0, r = k;

        // Step 1: build the sum of the very first window (first k elements)
        for (int i = 0; i < k; i++)
            sum += nums[i];
        prev = sum / k;   // average of the first window, my starting "best"

        // Step 2: slide the window across the rest of the array
        while (r < nums.length) {
            sum = sum + nums[r] - nums[l];   // add new element, remove old one
            l++;
            r++;
            curr = sum / k;
            if (curr > prev) prev = curr;    // keep the better average
        }

        return prev;
    }
}
```

**Walking through the logic:**
1. **Set up the first window.** I sum up the first `k` elements using a simple loop, and store its average in `prev`. This is my starting answer.
2. **Two pointers, `l` and `r`.** `l` marks the left edge of the window (the element that will be removed next), and `r` marks the element about to enter the window from the right.
3. **Slide one step at a time.** Every time I move the window forward by one, I subtract `nums[l]` and add `nums[r]` to `sum`. This keeps `sum` always equal to the sum of the current window, without ever looping through all `k` elements again.
4. **Compare and update.** After updating the sum, I compute `curr` (average of this new window) and check if it beats `prev`. If yes, `prev` gets updated.
5. **Return `prev`** once the window can't slide any further — it holds the best average found.

**Complexity:**
- Time: `O(n)` — each element is added to `sum` exactly once and removed exactly once, no matter how many windows there are.
- Space: `O(1)` — I only keep a running sum and a couple of pointers, no extra array.

## Brute Force vs Optimized

| | Brute Force | Sliding Window |
|---|---|---|
| Time | O(n * k) | O(n) |
| Space | O(1) | O(1) |
| Idea | Recompute sum for every window | Reuse previous window's sum, just adjust it |

So instead of redoing work for every window, the sliding window reuses the previous window's sum and just adjusts it. That's the whole trick.

## A Small Thing I Noticed

I used `double sum` right from the start, even though `nums[i]` values are `int`. This works fine and saves me a manual cast later when dividing by `k`. An alternative would be to keep `sum` as an `int` or `long` (since it's really counting whole numbers) and only convert to `double` at the very last step, when computing the average. Both work — it's just a matter of what feels more "correct": am I tracking a running integer sum, or a running average-in-progress? Something to think about for cleaner code next time.
