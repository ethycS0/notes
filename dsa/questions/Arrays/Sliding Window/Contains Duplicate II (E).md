Previous: [[Longest Mountain in Array (M)]] | Next: [[Minimum Absolute Difference (E)]]

You are given an integer array nums and an integer k, return true if there are two distinct indices i and j in the array such that nums[i] == nums[j] and abs(i - j) <= k, otherwise return false.

## Solution

- Create a Map
- Iterate over the array and find previous occurunce of nums[i]
- If found, check if j - i <= k
- if yes return true, if no set the map value to i (it->second = i) and continue

```C++
class Solution {
   public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        unordered_map<int, int> dict;
        bool result = false;

        for (int i = 0; i < nums.size(); i++) {
            auto it = dict.find(nums[i]);
            if (it != dict.end()) {
                if (i - it->second <= k) {
                    return true;
                } else {
                    it->second = i;
                }
            }

            dict.insert({nums[i], i});
        }

        return result;
    }
}
```