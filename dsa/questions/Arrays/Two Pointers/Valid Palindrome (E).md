Previous: [[Merge Sorted Array (E)]] | Next: [[Two Sum II - Input Array Is Sorted (M)]]

https://leetcode.com/problems/valid-palindrome/

A phrase is a palindrome if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Given a string , return  if it is a palindrome, or  otherwise.

## Solution

```C++
class Solution {
public:
    bool isPalindrome(string s) {
        string filtered = "";

        for (int i = 0; i < s.size(); i++) {
            if (isalnum(s[i])) {
                filtered += tolower(s[i]);
            }
        }

        int left = 0;
        int right = filtered.size() - 1;

        while (left < right) {
            if (filtered[left] != filtered[right]) {
                return false;
            }

            left += 1;
            right -= 1;
        }

        return true;
    }
};
```