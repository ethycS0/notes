Previous: [[Number of Islands (M)]] | Next: [[Squares of a Sorted Array (E)]]

You are given an integer array prices where prices[i] is the price of NeetCoin on the ith day.

You may choose a single day to buy one NeetCoin and choose a different day in the future to sell it.

Return the maximum profit you can achieve. You may choose to not make any transactions, in which case the profit would be 0.

## Solution

Check bounds first

l = h = prices[0]
profit = 0
max profit = 0

loop over array 1 -> n:
	if prices[i] < l
		l = prices[i]
		h = prices[i]
	else if prices[i] > h
		h = prices[i]
	
	profit = h - l
	max_profit = max(max_profit, profit)

return max_profit

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int lowest = prices[0];
        int highest = prices[0];
        int current_profit = 0;
        
        for(int i = 0; i < prices.size(); i++) {
            if(prices[i] < lowest) {
                lowest = prices[i];
                highest = prices[i];
            }

            if(prices[i] > highest) {
                highest = prices[i];

                if((highest - lowest) > current_profit) {
                    current_profit = highest - lowest;
                }
            }

        }

        return current_profit;
    }
}
```