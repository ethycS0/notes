Previous: [[Rotate Array (M)]] | Next: [[3Sum (M)]]

https://leetcode.com/problems/find-common-elements-between-two-arrays/

Given two 0-indexed integer arrays  and  of sizes  and , return an array  of size 2 containing the number of indices  where  exists in , and the number of indices  where  exists in .

## Solution
```Python
class Solution(object):
    def findIntersectionValues(self, nums1, nums2):
        num1 = set(nums1)
        num2 = set(nums2)
        i = j = 0
        for nums in nums1:
            if nums in num2:
                i += 1
        for nums in nums2:
            if nums in num1:
                j += 1
        return [i, j]
```