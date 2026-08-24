Previous: [[LRU Cache (M)]] | Next: [[Average of Levels in Binary Tree (E)]]

https://leetcode.com/problems/binary-search/

Given an array of integers  which is sorted in ascending order, and an integer , write a function to search  in . If  exists, then return its index. Otherwise, return .

## Solution
```C++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int result = -1;

        int l = 0;
        int r = nums.size() - 1;

        while (l <= r) {
            int mid = l + (r - l) / 2;
            int val = nums[mid];
            if (target < val) {
                r = mid - 1;
            } else if (target > val) {
                l = mid + 1;
            } else {
                result = mid;
                break;
            }
        }

        return result;
    }
};
```