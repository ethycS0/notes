Previous: [[Remove Duplicates from Sorted Array (E)]] | Next: [[Remove Duplicates from Sorted Array II (M)]]

https://leetcode.com/problems/remove-element/

Given an integer array  and an integer , remove all occurrences of  in  in-place. Return the number of elements in  which are not equal to .

## Solution
```C++
class Solution {
public:
int removeElement(vector<int>& nums, int val) {
        int k = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] != val) {
                nums[k++] = nums[i];
            }
        }
        return k;
    }
};
```