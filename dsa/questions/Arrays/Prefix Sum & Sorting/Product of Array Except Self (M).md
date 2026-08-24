Previous: [[Majority Element (E)]] | Next: [[Find Pivot Index (E)]]

https://leetcode.com/problems/product-of-array-except-self/

Given an integer array , return an array  such that  is equal to the product of all the elements of  except , calculated in O(n) without using division.

## Solution

```C++
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> answer;
        int last = 1;
        for(int i = 0; i < nums.size(); i++) {
            answer.push_back(last);
            last *= nums[i];
        }
        last = 1;
        for(int i = nums.size() - 1; i >= 0; i--) {
            answer[i] *= last;
            last *= nums[i];
        }
        return answer;
    }
};
```