Previous: [[Coin Change (M)]] | Next: [[Maximum Subarray (M)]]

You are given an integer n representing the number of steps to reach the top of a staircase. You can climb with either 1 or 2 steps at a time.

Return the number of distinct ways to climb to the top of the staircase.

## Solution

Again a DP problem. If we find the distinct ways it is required to climb the stairs from 1 -> n, we can find required distinct ways by adding the previous 2.

Example: 4 is required. We know 1 has 1 distinct way, 2 has 2, then 3 will have addition of previous 2 which is 3, now 4 will have addition of previous 2 which is 5.


1 -> 1 [1]
2 -> 2 [1, 1]  [2]
3 -> 3 [1, 1, 1]  [1, 2]  [2, 1]
4 -> 5 [1, 1, 1, 1]  [1, 1, 2]  [1, 2, 1]  [2, 1, 1]  [2, 2]

and so on

So we manage edge case 0, set array 1 -> 1, 2 -> 2 and then loop from 2 -> n

Here we will set ith value to arr[i - 1] + arr[i - 2] and find the result as so.

```C++
class Solution {
   public:
    int climbStairs(int n) {
        if (n < 1) {
            return 0;
        }

        vector<int> arr;

        arr.push_back(1);
        arr.push_back(2);

        int index = 2;

        while (index < n) {
            arr.push_back(arr[index - 1] + arr[index - 2]);
            index += 1;
        }

        return arr[n - 1];
    }
};

```