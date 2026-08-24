Previous: [[Find the Highest Altitude (E)]] | Next: [[Minimum Absolute Difference (E)]]

https://leetcode.com/problems/range-sum-query-immutable/

You are given an integer array nums, handle multiple queries of the following type:

    Calculate the sum of the elements of nums between indices left and right inclusive where left <= right.

Implement the NumArray class:

    NumArray(int[] nums) Initializes the object with the integer array nums.

    int sumRange(int left, int right) Returns the sum of the elements of nums between indices left and right inclusive (i.e. nums[left] + nums[left + 1] + ... + nums[right]).

## Solution

This is pretty simple. Instead of initializing an array of nums, we just create the DP array. This should be n + 1 size and initialized 0. Then we loop 0 -> n and and keep add nums to dp[i] and storing in dp[i+1]. Then we can just return dp[right + 1] - dp[left] in the sumRange function.

```C++
class NumArray {
private: 
    vector<int> dp;

public:
    NumArray(vector<int>& nums) {
        dp.resize(nums.size() + 1, 0);

        for (int i = 0; i < nums.size(); i++) {
            dp.at(i + 1) = dp.at(i) + nums.at(i);
        }
    }
    
    int sumRange(int left, int right) {
        return (dp.at(right + 1) - dp.at(left));
    }
}
```
