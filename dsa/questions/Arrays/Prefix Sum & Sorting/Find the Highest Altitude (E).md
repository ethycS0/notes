Previous: [[Find Pivot Index (E)]] | Next: [[Range Sum Query - Immutable (E)]]

https://leetcode.com/problems/find-the-highest-altitude/

There is a biker going on a road trip. You are given an integer array  of length  where  is the net gain in altitude between points  and . Return the highest altitude of a point.

## Solution
```C++
class Solution {
public:
    int largestAltitude(vector<int>& gain) {
        int maxi = 0;
        int cur = 0;
        for(int i = 0; i < gain.size(); i++) {
            cur += gain[i];
            maxi = max(cur, maxi);
        }
        return maxi;
    }
};
```