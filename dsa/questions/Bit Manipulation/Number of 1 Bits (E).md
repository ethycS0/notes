Previous: [[Single Number (E)]] | Next: [[Number of Even and Odd Bits (E)]]

https://leetcode.com/problems/number-of-1-bits/

Write a function that takes the binary representation of a positive integer and returns the number of set bits it has (also known as Hamming weight).

## Solution

```C++
class Solution {
public:
    int hammingWeight(int n) {
        int count = 0;
        while (n > 0) {
            n &= (n - 1);
            count++;
        }
        return count;
    }
};
```