Next: [[Missing Number (E)]]

Given an integer array nums, return true if any value appears more than once in the array, otherwise return false.

## Solutions 

- Quadratic solution: Comparing each to the later
- Linearithmic solution: Sort and compare i to i+1
- Linear solution: Check and insert into a hashmap

```C++
class Solution {
public:

    bool hasDuplicate(vector<int>& nums) {
        unordered_set<int> us = {};

        for(int i = 0; i < nums.size(); i++) {
            if(us.find(nums[i]) != us.end()) {
                return true;
            } else {
                us.insert(nums[i]);
            }

        }

        return false;
    }
};
```