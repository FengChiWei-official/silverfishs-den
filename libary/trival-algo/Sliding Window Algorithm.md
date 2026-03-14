---
tags:
  - type/permanent
  - status/evergreen
  - attr/principle
  - topic/learning
---
### Key Points
- [[Contiguous]]:  Subarray,yes; subsequence, no.
- [[Monotonicity]], when you expand your window, your state shift should be [[Monotonicity]].
	- i.e.  for (array, max subarray), you get a **minus integer**, your algorithm will collapse.
- [[Reversibility]]: operation for **in and out** should be low cost.

## Template
### Principles
1. initialize the window outside the loop
2. update the window, when window expand, **not failed**.
[[Template of Sliding Window]]
[[Template of Fixed Window]]

---
## **Related**：