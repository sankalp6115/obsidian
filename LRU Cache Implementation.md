A data structure where elmts are put for quick access, then when it fill up , the elmt that was last used is removed.

- [x] Get 
- [x] Put
- [x] Capacity exceeded , evict
- [x] get and put update order
- [x] get and put O(1)
- [x] Implement the cache without using Java's built-in linked-list implementation.
- [x] Handle updates to existing keys correctly.
- [x] Evict the correct entry when capacity is exceeded.
- [x] Test capacity-one caches, repeated accesses, updates, and evictions.
- [x] Pass the [LeetCode 146: LRU Cache](https://leetcode.com/problems/lru-cache/) test cases.
- [x] Explain why both operations achieve O(1) average time.

DLL:
- Head, tail
- key:val
- evict()
- addToLast()
- updateToLast()