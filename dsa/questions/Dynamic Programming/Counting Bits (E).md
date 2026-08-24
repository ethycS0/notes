Previous: [[Maximum Subarray (M)]] | Next: [[Letter Case Permutation (M)]]

https://leetcode.com/problems/counting-bits/

Given an integer n, count the number of 1's in the binary representation of every number in the range [0, n].

Return an array output where output[i] is the number of 1's in the binary representation of i.


## Solution

This question is also interesting. Basically, its an algorithm where a specific pattern repeats as such:

0 -> 0
1 -> 1
2 -> 1  (0th + 1)
3 -> 2  (1st + 1)
4 -> 1  (0th + 1)
5 -> 2  (1st + 1)
6 -> 2  (2nd + 1)
7 -> 3  (3rd + 1)
8 -> 1  (0th + 1)
and so on

So basically, we have 2 indices to keep track of x and y.
We create a DP array and push_back 0 and 1 as prerequisites.
x will start at 0 and y will at 2.

Now we loop from i -> n
We push_back 1 + dp[x], increment x and do a check
if ( x == y) -> y += y and x = 0
And we loop to next.


This is how we recreate the above alogrithm.

```C++
class Solution {
   public:
    vector<int> countBits(int n) {
        if (n < 1) {
            return {0};
        }

        vector<int> r;
        r.push_back(0);
        r.push_back(1);

        int x = 0;
        int y = 2;

        for (int i = 1; i < n; i++) {
            r.push_back(1 + r[x]);
            x += 1;
            if (x == y) {
                y += y;
                x = 0;
            }
        }

        return r;
    }
};

```
