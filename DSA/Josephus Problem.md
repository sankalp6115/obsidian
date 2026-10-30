How It Works

- **Setup:** n people stand in a circle, numbered from 1 to n.
- **The Rule:** Starting from a chosen person, you count around the circle and eliminate every k-th person.
- **Continuation:** The circle shrinks as people are removed, and the count continues from the person immediately after the last eliminated individual until only **one survivor** is left.

```java
class Solution {
	public int findTheWinner(int n, int k) {
		if(n == 1) return 1;
		return (findTheWinner(n-1,k) + k - 1) % n + 1;
	}
}
```


Leetcode: https://leetcode.com/problems/find-the-winner-of-the-circular-game/