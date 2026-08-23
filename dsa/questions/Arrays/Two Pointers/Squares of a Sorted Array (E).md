Previous: [[Best Time to Buy and Sell Stock (E)]] | Next: [[3Sum (M)]]

You are given an integer array nums sorted in non-decreasing order, return an array of the squares of each number sorted in non-decreasing order.

## Solution

- Linearithmic: Square and sort array
- Linear: There is a midpoint method bt too much hassle. Just compare edges, so abs(l) vs abs(r) and insert greater at the end.

```C++
class Solution {
   public:
    vector<int> sortedSquares(vector<int>& nums) {
        int n = nums.size();
        vector<int> res(n);
        int l = 0, r = n - 1;

        for (int k = n - 1; k >= 0; k--) {
            if (abs(nums[l]) > abs(nums[r])) {
                res[k] = nums[l] * nums[l];
                l++;
            } else {
                res[k] = nums[r] * nums[r];
                r--;
            }
        }

        return res;
    }
}
```