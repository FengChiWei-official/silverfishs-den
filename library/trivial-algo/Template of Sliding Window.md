---
tags:
  - type/lit
  - topic/learning
  - status/archive
source: 《卡片盒笔记法》 by Sönke Ahrens
---

## Text

``` cpp

class Solution {

public:
	int longestSubarray(vector<int>& nums, int limit) {
		
		// window's state
		deque<int> qM;
		deque<int> qm;
		// answer
		int ans = 0;
		
		int l = 0;
		// 1. right expand
		for (int r = 0; r < nums.size(); r++) {
		
		
			
			// // update state1 Max queue
			while (!qM.empty() && nums[qM.back()] <= nums[r]) {
				qM.pop_back();
			}
			qM.push_back(r);
			
			
			// // update state1 Max queue
			while (!qm.empty() && nums[qm.back()] >= nums[r]) {
				qm.pop_back();
			}
			qm.push_back(r);
			
			
			// 3. left shrinking
			while (
				!qM.empty() && !qm.empty() && 
				nums[qM.front()] - nums[qm.front()] > limit
			) {
			
				if (qM.front() == l) qM.pop_front();
				if (qm.front() == l) qm.pop_front();
				l++;
			
			}

			// update answer
			// window size is **r - l + 1**
			
			ans = max(ans, r - l + 1);
			
			}
		return ans;
	}
};
```

---

## Thoughts

1. Sliding Window doesn't need initialize the state variable.
2. `r` is a variable growing step by step, so `for` statement is suitable solution.

