Previous: [[Product of Array Except Self (M)]] | Next: [[Find the Highest Altitude (E)]]

https://leetcode.com/problems/find-pivot-index/

Given an array of integers , calculate the pivot index of this array. The pivot index is the index where the sum of all the numbers strictly to the left of the index is equal to the sum of all the numbers strictly to the index's right.

## Solution
```C++
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int left = 0;
        int max = nums.size();
        int right = 0;

        for (int i = 1; i < max; i++) {
            right += nums[i];
        }

        for (int i = 1; i < max; i++) {
            if (left == right) {
                return i - 1;
            }

            left += nums[i - 1];
            right -= nums[i];
        }

        if(left == 0) {
            return max - 1;
        }

        return -1;
    }
};
```