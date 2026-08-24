Previous: [[Best Time to Buy and Sell Stock (E)]] | Next: [[Maximum Average Subarray I (E)]]

https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/

You are given an integer array  where  is the price of a given stock on the -th day. On each day, you may decide to buy and/or sell the stock. Find and return the maximum profit you can achieve.

## Solution

```C++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int lowest = prices[0];
        int highest = prices[0];
        int profit_per = 0;
        int profit = 0;
        for(int i = 0; i < prices.size(); i++) {
            if(prices[i] < lowest) {
                lowest = prices[i];
                highest = 0;
                profit += profit_per;
                profit_per = 0;
            } else {
                if(prices[i] > highest) {
                    highest = prices[i];
                    profit_per = prices[i] - lowest;
                } else {
                    profit += profit_per;
                    profit_per = 0;
                    lowest = prices[i];
                    highest = 0;
                }
            }
        }
        profit += profit_per;

        return profit;
    }
};
```