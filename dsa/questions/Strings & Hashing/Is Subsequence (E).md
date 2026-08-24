Previous: [[Reverse Vowels of a String (E)]] | Next: [[Greatest Common Divisor of Strings (E)]]

https://leetcode.com/problems/is-subsequence/

Given two strings s and t, return true if s is a subsequence of t, or false otherwise.

A subsequence of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., "ace" is a subsequence of "abcde" while "aec" is not).

## Solution

```C++
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int loc = 0;
        for (int i = 0; i < s.size(); i++) {
            int tmp = t.find(s[i], loc);
            if (tmp == string::npos) {
                return false;
            }
            loc = tmp + 1;
        }
        return true;
    }
};
```