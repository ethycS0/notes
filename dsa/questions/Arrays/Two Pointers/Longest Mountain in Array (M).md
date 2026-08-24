Previous: [[3Sum (M)]] | Next: [[Squares of a Sorted Array (E)]]

https://leetcode.com/problems/longest-mountain-in-array/

You may recall that an array arr is a mountain array if and only if:

    arr.length >= 3
    There exists some index i (0-indexed) with 0 < i < arr.length - 1 such that:
        arr[0] < arr[1] < ... < arr[i - 1] < arr[i]
        arr[i] > arr[i + 1] > ... > arr[arr.length - 1]

Given an integer array arr, return the length of the longest subarray, which is a mountain. Return 0 if there is no mountain subarray.

## Solution

max_len = 0

loop i from 1 -> n - 2:
    if arr[i-1] < arr[i] and arr[i] > arr[i+1]:  // valid peak
        l = i, r = i
        
        while l > 0 and arr[l-1] < arr[l]:       // expand left
            l--
            
        while r < n - 1 and arr[r] > arr[r+1]:   // expand right
            r++
            
        max_len = max(max_len, r - l + 1)
        i = r                                    // jump i to end of current mountain

return max_len

```C++
class Solution {
public:
    int longestMountain(vector<int>& arr) {
        int result = 0;
        int front = 1;
        int forward, backward = 0;
        if (arr.size() < 3) {
            return 0;
        }

        while (front < arr.size()) {
            while (front < arr.size() && arr[front] > arr[front - 1]) {
                front += 1;
                forward += 1;
            }

            if (forward == 0) {
                backward = 0;
                front += 1;
                continue;
            }

            while (front < arr.size() && arr[front] < arr[front - 1]) {
                front += 1;
                backward += 1;
            }

            if (backward == 0) {
                forward = 0;
                front += 1;
                continue;
            }

            result = max(result, (forward + backward + 1));
            forward = 0;
            backward = 0;
        }

        return result;
    }
};
```
