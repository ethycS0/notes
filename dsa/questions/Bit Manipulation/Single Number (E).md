Previous: [[Merge Strings Alternately (E)]] | Next: [[Number of 1 Bits (E)]]

https://leetcode.com/problems/single-number/

You are given a non-empty array of integers nums. Every integer appears twice except for one.

Return the integer that appears only once.

You must implement a solution with O(n)O(n) runtime complexity and use only O(1)O(1) extra space.

## Solution

- This is a trick of XOR
- a ^ a = 0
- Hence all numbers in the array are XORed. 
- Every duplicate cancels each other and at the end we get 
- b ^ 0 = b
- Hence we find the single number b

```C++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;

        for (auto n : nums) {
            res ^= n;
        }

        return res;
    }
};

```
