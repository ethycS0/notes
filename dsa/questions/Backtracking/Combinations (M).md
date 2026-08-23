Previous: [[Subsets (M)]] | Next: [[Permutations (M)]]

You are given two integers n and k, return all possible combinations of k numbers chosen from the range [1, n].

You may return the answer in any order.

## Solution

This will be done with DFS and backtracking. Essentially we need to create combinations of k integers from 1 -> n.

The backtrack function should do the following:

Append initial integer and call itself back. The next call should now insert the next indexed number. This till the limit of size k -> append and pop_back. Next iteration it should insert the next index. So basically

Example: n = 3 k = 2

Here, we will call backtrack, it should take index, current, n, k, result.

We start by creating an insertion into result. Current vector will be inserted inside the result only when size == k, else it goes to the next part.

This is the actual loop, we loop from index to n here we create the first c inserting 0th index which in this case will be 1, then we backtrack in a similar fashion to the backtrack in [[Subsets (M)]] question.

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
}
```