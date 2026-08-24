Previous: [[Remove Element (E)]] | Next: [[Merge Sorted Array (E)]]

https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/

Given an integer array  sorted in non-decreasing order, remove some duplicates in-place such that each unique element appears at most twice. Return  after placing the final result in the first  slots of .

## Solution
```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
       int k = 0;
       bool threepeat = false;
       for(int i = 0; i < nums.size() - 1; i++) {
        if(nums[i] != nums[i+1]) {
            nums[k++] = nums[i];
            threepeat = false;
        } else {
            if(!threepeat) {
                nums[k++] = nums[i];
                threepeat = true;
            } else {
                continue;
            }
        }
       }
       //if(!threepeat) {
        nums[k++] = nums[nums.size() - 1];
       //}
       return k;
    }
};
```