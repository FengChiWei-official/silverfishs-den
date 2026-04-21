---
tags:
  - type/permanent
  - attr/principle
  - topic/learning
  - status/evergreen
aliases:
  - LCS-like DP
---

## qwq

Index of DP is current prefix length of each Sequence.
The transfer logic is 
1. Check `A[i] == B[j]`
	1. case yes: `A[0:i]` and `B[0:j]` have no need to be consider.
	2. case no: `A[i], B[j]`should consider their Prefix.
		1. (`A[0:i]` and `B[0:j-1]`) or (`A[0:i-1]` and `B[0:j]`)

---
## **Related**：

### Example
standard
[[1143. Longest Common Subsequence]]