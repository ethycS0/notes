Previous: [[Minimum Absolute Difference (E)]] | Next: [[Reverse Words in a String (M)]]

https://leetcode.com/problems/find-nearest-point-that-has-the-same-x-or-y-coordinate/

You are given two integers,  and , which represent your current location on a 2D grid, and an array  where . Return the index of the valid point with the smallest Manhattan distance from your current location.

## Solution
```Python
class Solution(object):
    def nearestValidPoint(self, x, y, points):
        MD = {}
        anss = []
        for i in range(len(points)):
            n = points[i]
            m = n[0]
            l = n[1] 
            if m == x or l == y:
                ans = abs(x - m) + abs(y - l)
                MD[i] = ans
        if MD == {}:
            return -1
        else:    
            z = min(MD.values())
           
            for key, value in MD.items():
                if value == z:
                    anss.append(key)
            p = min(anss)
            for key in anss:
                if key == p:
                    return key
```