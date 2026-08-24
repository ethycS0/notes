Previous: [[Valid Palindrome (E)]] | Next: [[Move Zeroes (E)]]

https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/

Given a 1-indexed array of integers  that is already sorted in non-decreasing order, find two numbers such that they add up to a specific  number.

## Solution
```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left = 0;
        int right = numbers.size() - 1;
        while(left < right) {
            if(target == (numbers[left] + numbers[right])) {
                return {left + 1, right + 1};
            } else if(target > numbers[left] + numbers[right]) {
                left += 1;
            } else {
                right -= 1;
            }
        }

        return {0};
    }
};
```