Previous: [[Is Subsequence (E)]] | Next: [[Merge Strings Alternately (E)]]

https://leetcode.com/problems/greatest-common-divisor-of-strings/

For two strings s and t, we say "t divides s" if and only if s = t + t + t + ... + t + t (i.e., t is concatenated with itself one or more times).

Given two strings str1 and str2, return the largest string x such that x divides both str1 and str2.

## Solution

```C++
class Solution {
public:
    bool isValid(string str, string result, int len, int gcd) {
        int j = 0;
        for (int i = 0; i < len; i++) {
            if(j > gcd - 1) {
                j = 0;
            }
            if(str[i] != result[j]) {
                return false;
            }
            j += 1;
        }
        return true;
    }

    string gcdOfStrings(string str1, string str2) {
        std::string result = "";
        int len1 = str1.size();
        int len2 = str2.size();
        int gcd = min(len1, len2);

        while (true) {
            if (len1 % gcd == 0 && len2 % gcd == 0) {
                break;
            }
            gcd -= 1;
        }

        for (int i = 0; i < gcd; i++) {
            if (str1[i] != str2[i]) {
                break;
            }
            result += str1[i];
        }

        if(isValid(str1, result, len1, gcd) == false || isValid(str2, result, len2, gcd) == false) {
            return "";
        }

        return result;
    }
};
```