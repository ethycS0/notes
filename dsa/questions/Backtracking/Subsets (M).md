Previous: [[Letter Case Permutation (M)]] | Next: [[Combinations (M)]]

Given an array nums of unique integers, return all possible subsets of nums.

The solution set must not contain duplicate subsets. You may return the solution in any order.

## Solution

Another problem requiring BFS like backtracking. Here, we will instead use DFS.

Here the algorithm requires a backtrack function that takes in an index, the nums array, an int array c and the resultant vectors of vector ints r.


The working of this DFS approach is as follows:

We take the vector c and push it back of r. This at the start will fullfill out requirement of empty subset.

Then we iterate from index which should be 0 at the start to nums.size() and we push_back the ith integer into c and backrack again.

This structure will essetially do the following


Example: [1. 2. 3]

r -> [[]]

c in first backtrack -> [1]
c in second backtrack  -> [1, 2]
c in third backtrack -> [1, 2, 3]

r -> [[], [1], [1, 2], [1, 2, 3] ]

3rd backtrack returns cuz index = size
2nd backtrack pop_back -> [1] -> next iteration -> [1, 3] -> 3rd backtrack again -> return
1st backtrack pop_back -> [] -> next iteration -> [2]
then again 2nd backtrack [2, 3] -> return

r -> [[], [1], [1, 2], [1, 2, 3], [2], [2, 3] ]

and finally 3rd iteration of backtrack 1 will provide [3]

r -> [[], [1], [1, 2], [1, 2, 3], [2], [2, 3], [3] ]


```C++
class Solution {
   public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> r;
        vector<int> c;

        backtrack(0, nums, c, r);

        return r;
    }

   private:
    void backtrack(int index, vector<int>& nums, vector<int>& c, vector<vector<int>>& r) {
        r.push_back(c);

        for (int i = index; i < nums.size(); i++) {
            c.push_back(nums[i]);
            backtrack(i + 1, nums, c, r);
            c.pop_back();
        }
    }
}
```
