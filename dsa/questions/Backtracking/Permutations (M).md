Previous: [[Combinations (M)]] | Next: [[Middle of the Linked List (E)]]

https://leetcode.com/problems/permutations/

Given an array nums of unique integers, return all the possible permutations. You may return the answer in any order.


## Solution

Yeah, requires a fresh brain

```C++
class Solution {
   public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> r;
        backtrack(0, nums, r);
        return r;
    }

   private:
    void backtrack(int index, vector<int>& nums, vector<vector<int>>& r) {
        if (index == nums.size()) {
            r.push_back(nums);
            return;
        }

        for (int i = index; i < nums.size(); i++) {
            swap(nums[index], nums[i]);     
            backtrack(index + 1, nums, r); 
            swap(nums[index], nums[i]);   
        }
    }
};

```
