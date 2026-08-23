Previous: [[Climbing Stairs (E)]] | Next: [[Counting Bits (E)]]

Given an array of integers nums, find the subarray with the largest sum and return the sum.

A subarray is a contiguous non-empty sequence of elements within an array.

## Solution

This one is deceptively simple with dynamic programming. We just iterate and keep the maximum value at the ith index. Basically,

We start with max = nums[0]

Then we iterate from 1 -> n

every turn, ith value will be maximum of either nums[i] or nums[i] + nums[i - 1]. This allows us to get the local maximum. Then we inplace replace nums[i] with max of nums[i] and (nums[i] + nums[i - 1]). We keep an ongoing track of max with max is nums[i] vs max.

```C++
class Solution {
   public:
    int maxSubArray(vector<int>& nums) {
        int max_num = nums[0];

        for (int i = 1; i < nums.size(); i++) {
            nums.at(i) = max(nums[i], (nums[i] + nums[i - 1]));
            max_num = max(nums[i], max_num);
        }

        return max_num;
    }
};

```