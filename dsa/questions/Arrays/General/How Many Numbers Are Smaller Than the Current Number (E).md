Previous: [[Two Sum (E)]] | Next: [[Minimum Time Visiting All Points (E)]]

Given the array nums, for each nums[i] find out how many numbers in the array are smaller than it. That is, for each nums[i] you have to count the number of valid j's such that j != i and nums[j] < nums[i].

Return the answer in an array.

## Solution

- Quadratic: Iterate twice, compare and increment number ([i] < [j]), then finally store in result array at i
- Linearithmic: Create sorted copy, insert into a based rank map, iterate over original and find rank

```C++
class Solution {
public:
    vector<int> smallerNumbersThanCurrent(vector<int>& nums) {
        vector<int> sorted = nums;
        sort(sorted.begin(), sorted.end());

        unordered_map<int, int> rank;
        for (int i = 0; i < sorted.size(); ++i) {
            if (rank.find(sorted[i]) == rank.end()) {
                rank[sorted[i]] = i;
            }
        }

        for (int i = 0; i < nums.size(); ++i) {
            nums[i] = rank[nums[i]];
        }

        return nums;
    }
};
```