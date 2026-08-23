Previous: [[Combinations (M)]] | Next: [[Middle of the Linked List (E)]]

Given an array nums of unique integers, return all the possible permutations. You may return the answer in any order.


## Solution

Yeah, requires a fresh brain

```C++
class Solution {
   public:
    vector<vector<int>> combine(int n, int k) {
        vector<int> current;
        vector<vector<int>> result;

        backtrack(1, current, n, k, result);

        return result;
    }

   private:
    void backtrack(int index, vector<int>& c, int n, int k, vector<vector<int>>& r) {
        if (c.size() == k) {
            r.push_back(c);
            return;
        }

        for (int i = index; i <= n; i++) {
            c.push_back(i);
            backtrack(i + 1, c, n, k, r);
            c.pop_back();
        }
    }
};
```
