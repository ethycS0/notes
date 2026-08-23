Previous: [[Contains Duplicate II (E)]] | Next: [[Minimum Size Subarray Sum (M)]]

Given an array of distinct integers arr, find all pairs of elements with the minimum absolute difference of any two elements.

Return a list of pairs in ascending order(with respect to pairs), each pair [a, b] follows

    a, b are from arr
    a < b
    b - a equals to the minimum absolute difference of any two elements in arr

## Solution

- Set min_abs_diff to max value
- Sort array
- nums[i + 1] - nums[i] = min_abs_diff
- keep updating minimums
- Append pairs that satisfy min_abs_diff to array, if min_abs_diff updates, clear the result array and start again
- return res

```C++
class Solution {
public:
    vector<vector<int>> minimumAbsDifference(vector<int>& arr) {
        sort(arr.begin(), arr.end());
        int lowest = arr[1] - arr[0];
        vector<vector<int>> result;

        for(int i = 1; i < arr.size(); i++) {
            if(arr[i] - arr[i - 1] == lowest) {
                result.push_back({arr[i - 1], arr[i]});
            } else if(arr[i] - arr[i - 1] < lowest) {
                result.clear();
                lowest = arr[i] - arr[i - 1];
                result.push_back({arr[i - 1], arr[i]});
            }
        }

        return result;
    }
};
```