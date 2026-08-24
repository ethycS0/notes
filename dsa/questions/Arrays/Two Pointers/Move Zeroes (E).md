Previous: [[Two Sum II - Input Array Is Sorted (M)]] | Next: [[Contains Duplicate II (E)]]

https://leetcode.com/problems/move-zeroes/

Given an integer array , move all 's to the end of it while maintaining the relative order of the non-zero elements in-place.

## Solution
```C++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int zeroes = 0;
        for(int i = 0; i < nums.size();) {
            if(nums[i] == 0) {
                nums.erase(nums.begin() + i);
                zeroes += 1;
            } else {
                i += 1;
            }
        }

        for(int i = 0; i < zeroes; i++) {
            nums.push_back(0);
        }
    }
};
```