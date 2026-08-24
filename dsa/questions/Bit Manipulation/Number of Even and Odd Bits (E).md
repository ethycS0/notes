Previous: [[Number of 1 Bits (E)]] | Next: [[Add Two Integers (E)]]

https://leetcode.com/problems/number-of-even-and-odd-bits/

You are given a positive integer . Let  denote the number of even indices in the binary representation of  with a value of , and  denote the number of odd indices with value . Return an array .

## Solution
```Python
class Solution(object):
    def evenOddBit(self, n):
        num = bin(n)[2:]
        l = len(num)
        x = 0
        y = 0
        for i in range(l):
            if ((l-i-1)%2) == 0:
                if num[i] == '1':
                    x += 1
            else:
                if num[i] == '1':
                    y += 1
        return [x, y]
        

```