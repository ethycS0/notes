Previous: [[Minimum Absolute Difference (E)]] | Next: [[Single Number (E)]]

You are given an array of positive integers nums and a positive integer target, return the minimal length of a subarray whose sum is greater than or equal to target. If there is no such subarray, return 0 instead.

A subarray is a contiguous non-empty sequence of elements within an array.

## Solution

- Check edge cases for size 0 and 1
- Initialize sum to nums[0], f to 1, b to 0 and min_len to nums.size() + 1
- Loop while f < size
- Increment f and sum += nums[f] till f is within bounds and sum < target
- Do the same for b, while b <= f and sum >= target
- This gives current iteration smallest subarray with (f - b + 1), find min between min_len and that.
- Do this and end loop. If min_len is nums.size() + 1, return 0 else return min_len

```C++
class Solution {
   public:
    int minSubArrayLen(int target, vector<int>& nums) {
        if (nums.empty()) {
            return 0;
        }

        if(nums.size() < 2 && nums[0] == target) {
            return 1;
        }

        int min_len = nums.size() + 1;
        int sum = nums[0];
        int f = 1, b = 0;

        while (f < nums.size()) {
            while (sum < target && f < nums.size()) {
                sum += nums[f];
                f += 1;
            }

            while (sum >= target && b <= f) {
                sum -= nums[b];
                b += 1;
            }

            min_len = min(min_len, (f - b + 1));
        }

        return (min_len != nums.size() + 1) ? min_len : 0;
    }
}
```