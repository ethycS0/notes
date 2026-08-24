Previous: [[Best Time to Buy and Sell Stock II (M)]] | Next: [[Can Place Flowers (E)]]

https://leetcode.com/problems/maximum-average-subarray-i/

You are given an integer array  consisting of  elements, and an integer . Find a contiguous subarray whose length is equal to  that has the maximum average value and return this value.

## Solution
```C++
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        double max_avg = -10000;
        double avg = 0;
        for (double j = 0; j < k; j++) {
            avg += nums[j];
        }
        max_avg = avg;
        for (double i = k; i < nums.size(); i++) {
            avg -= nums[i - k];
            avg += nums[i];
            max_avg = max(avg, max_avg);
        }
        return max_avg / k;
    }
};
```