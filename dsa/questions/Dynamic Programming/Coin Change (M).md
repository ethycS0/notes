Previous: [[Single Number (E)]] | Next: [[Climbing Stairs (E)]]

You are given an integer array coins representing coins of different denominations (e.g. 1 dollar, 5 dollars, etc) and an integer amount representing a target amount of money.

Return the fewest number of coins that you need to make up the exact target amount. If it is impossible to make up the amount, return -1.

You may assume that you have an unlimited number of each coin.

## Solution

- This is a dynamic programming question. So I will break the question into parts.

We require to reach x amount. So what we can do is, iterate towards x such that, we get all amounts from 0 to x built with the given coins. This will allow us to know the minimum amount.

Example: If coins are [1, 5] and required amount is 12, if we know that amount 5 requires 1 coin, 10 requires 2 coins that 12 require 4. Ofcourse here we will be able to find all coins required for all amounts from 0 being 0, 1 being 1, 2 being 2, 5 being 1, being 2 and so on.

We can do this by creating and array of amount + 1 size so that we can store from 0 -> amount amounts coins required. Initialize it to max size (amounts + 1) so that we have the case of impossible amount with given denominations. Then we will initialize 0th index to 0 since 0 coins required.

Then we loop from 1 -> amount. Here we have a nested loop for all coins.

We can now check if the given amount which is `i` is greater than the current coin denomination c by doing `if ((i - c) >= 0)`. If this is true, that means that we can use a previously computed value. This can be done by finding the minimum between the current stored value at i (might be other coins results so we just try to find the minimum) and 1 coin + the previously computed value that is at dp[i - c]. In short: `dp.at(i) = min(dp.at(i), 1 + dp.at(i - c));
`
Example: If given denominations are [1, 2, 5], we want to find for 7, we iterate

So 7 - c:   for 1 = 7 - 1 = 6
We have previously seen 6 requires 2 coins, so 7 would require 3, this is current minimum
Next coin iteration, we check for 2 .. and for 5. For 5, i - c = 2, we see that 2 required 1 and +1 = 2 so 7 requires 2. That is how we reach the amount.

```C++
class Solution {
   public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);
        dp.at(0) = 0;

        for (int i = 1; i <= amount; i++) {
            for (auto c : coins) {
                if (i - c >= 0) {
                    dp.at(i) = min(dp.at(i), 1 + dp.at(i - c));
                }
            }
        }

        return (dp.at(amount) != amount + 1) ? dp.at(amount) : -1;
    };
};

```