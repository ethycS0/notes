Previous: [[Contains Duplicate (E)]] | Next: [[Find All Numbers Disappeared in An Array (E)]]

https://leetcode.com/problems/missing-number/

Given an array nums containing n integers in the range 0, n without any duplicates, return the single number in the range that is missing from nums.

Follow-up: Could you implement a solution using only O(1) extra space complexity and O(n) runtime complexity?

## Solutions

- Linearithmic: Sort and check missing
- Linear: Add all values till n and subtract from n! {n(n+1)/2}

```C++
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int n = nums.size();

        int x = (n * (n + 1)) / 2;
        int y = 0;

        for(int i = 0; i < n; i++) {
            y += nums[i];
        }

        return x - y;
        
    }
};
```
