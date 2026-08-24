Previous: [[Greatest Common Divisor of Strings (E)]] | Next: [[Single Number (E)]]

https://leetcode.com/problems/merge-strings-alternately/

You are given two strings  and . Merge the strings by adding letters in alternating order, starting with .

## Solution

```C++
class Solution {
public:
    string mergeAlternately(string word1, string word2) {
        std::string answer;
        int len1 = word1.size();
        int len2 = word2.size();
        int i = 0;
        if(len1 <= len2) {
            for(i = 0; i < len1; i++ ) {
                answer.append(word1, i, 1);
                answer.append(word2, i, 1);
            }
            answer.append(word2, i, len2-i);
        } else {
            for(i = 0; i < len2; i++ ) {
                answer.append(word1, i, 1);
                answer.append(word2, i, 1);
            }
            answer.append(word1, i, len1-i);
        }
    return answer;
    }
};
```