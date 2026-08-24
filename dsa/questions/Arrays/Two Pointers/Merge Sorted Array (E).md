Previous: [[Remove Duplicates from Sorted Array II (M)]] | Next: [[Valid Palindrome (E)]]

https://leetcode.com/problems/merge-sorted-array/

You are given two integer arrays  and , sorted in non-decreasing order, and two integers  and , representing the number of elements in  and  respectively. Merge  into  as one sorted array in-place.

## Solution

```C++
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int total = m + n - 1;
        for (int i = total; i >= 0; i--) {
            if(n == 0){
                nums1[i] = nums1[m-1];
                m -= 1;
                continue;
            } else if (m == 0) {
                nums1[i] = nums2[n-1];
                n -= 1;
                continue;
            }
            if(nums1[m-1] > nums2[n-1]) {
                nums1[i] = nums1[m-1];
                m -= 1;
            } else {
                nums1[i] = nums2[n-1];
                n -= 1;
            }
        }
    }
};
```