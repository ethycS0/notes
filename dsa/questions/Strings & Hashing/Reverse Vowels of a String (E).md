Previous: [[Valid Anagram (E)]] | Next: [[Is Subsequence (E)]]

https://leetcode.com/problems/reverse-vowels-of-a-string/

Given a string , reverse only all the vowels in the string and return it.

## Solution
```C++
class Solution {
public:
    string reverseVowels(string s) {
        string vowels = "aeiou";
        int size = s.size();
        vector<int> locs;
        for(int i = 0; i < size; i++) {
            for(int j = 0; j < vowels.size(); j++) {
                if(tolower(s[i]) == vowels[j]) {
                    locs.push_back(i);
                }
            }
        }

        int found = locs.size();
        
        for(int i = 0; i < (found / 2); i++) {
            string tmp = "";
            tmp = s[locs[i]];
            s[locs[i]] = s[locs[found - i - 1]];
            s[locs[found - i - 1]] = tmp[0]; 
        }
        return s;
    }
};
```