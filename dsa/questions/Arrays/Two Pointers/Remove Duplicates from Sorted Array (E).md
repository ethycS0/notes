Previous: [[Squares of a Sorted Array (E)]] | Next: [[Remove Element (E)]]

https://leetcode.com/problems/remove-duplicates-from-sorted-array/

Given an integer array  sorted in non-decreasing order, remove the duplicates in-place such that each unique element appears only once. Return the number of unique elements .

## Solution
```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int k = 0;
        for(int i = 0; i < (nums.size() - 1); i++) {
            if(nums[i] != nums[i+1]) {
                nums[k++] = nums[i];
            }
        }
        nums[k++] = nums[nums.size() - 1];
        return k;
    }
};
```