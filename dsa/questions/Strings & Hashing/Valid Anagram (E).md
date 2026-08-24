Previous: [[Reverse Words in a String (M)]] | Next: [[Reverse Vowels of a String (E)]]

https://leetcode.com/problems/valid-anagram/

Given two strings  and , return  if  is an anagram of , and  otherwise.

## Solution
```C++
class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.size() != t.size()) {
            return false;
        }

        unordered_map<char, int> m;

        for (int i = 0; i < s.size(); i++) {
            if (m.find(s[i]) != m.end()) {
                m[s[i]] += 1;
            } else {
                m.insert({s[i], 1});
            }
        }

        for (int i = 0; i < t.size(); i++) {
            if (m.find(t[i]) != m.end()) {
                m[t[i]] -= 1;
                if (m[t[i]] < 1) {
                    m.erase(t[i]);
                }
            } else {
                return false;
            }
        }

        return true;
    }
};
```