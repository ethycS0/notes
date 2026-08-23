Previous: [[Find All Numbers Disappeared in An Array (E)]] | Next: [[How Many Numbers Are Smaller Than the Current Number (E)]]

You are given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

You can return the answer in any order.

## Solution

- Quadratic Solution: Iterate twice and check i + j = target
- Linear: Insert into a map, iterate again and check r.find(target - [i]), get j and return i, j
- Linearithmic: Sort, two pointer i and j, if i + j: (> target : j--), (< target: i++) and return i, j

```C++
class Solution {
   private:
    unordered_map<int, int> arr;

   public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++) {
            int result = target - nums[i];
            auto it = arr.find(result); 

            if(it != arr.end()) {
                return {it->second, i};
            }

            arr.insert({nums[i], i});
        }

        return {0, 0};
    }
}
```