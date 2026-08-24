Previous: [[Can Place Flowers (E)]] | Next: [[Majority Element (E)]]

https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/

There are  kids with candies. You are given an integer array , where each  represents the number of candies the -th kid has, and an integer . Return a boolean array  of length , where  is  if giving the -th kid all  makes them have the greatest number of candies among all kids.

## Solution
```C++
class Solution {
public:
    vector<bool> kidsWithCandies(vector<int>& candies, int extraCandies) {
        vector<bool> result;
        int great = 0;
        for(int i = 0; i < candies.size(); i++) {
             for(int j = 0; j < candies.size(); j++) {
                if(candies[i] + extraCandies < candies[j]) {
                    great = 0;
                    break;
                }
                great = 1;
            }
            if(great == 1) {
                result.push_back(true);
            } else {
                result.push_back(false);
            }
        }
        return result;
    }
};
```