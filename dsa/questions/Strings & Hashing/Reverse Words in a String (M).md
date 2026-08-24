Previous: [[Find Nearest Point That Has the Same X or Y Coordinate (E)]] | Next: [[Valid Anagram (E)]]

https://leetcode.com/problems/reverse-words-in-a-string/

Given an input string , reverse the order of the words. A word is defined as a sequence of non-space characters.

## Solution
```C++
class Solution {
public:
    string reverseWords(string s) {
        std::vector<string> words;
        int start = 0;
        while (start < s.size() && s[start] == ' ')
            start++;
        int end = s.size() - 1;
        while (end >= 0 && s[end] == ' ')
            end--;
        while (true) {
            int pos = s.find(" ", start);
            if (pos == string::npos) {
                words.push_back(s.substr(start, pos));
                break;
            }
            int len = pos - start;
            words.push_back(s.substr(start, len));
            start = pos + 1;
        }

        std::string result = "";
        for (int i = words.size() - 1; i >= 0; i--) {
            string word = words[i];
            if (word.size() < 1) {
                continue;
            }
            for (int j = 0; j < word.size(); j++) {
                if (word[j] == ' ') {
                    word.erase(j, 1);
                }
            }
            result += word;
            if (i > 0) {
                result += " ";
            }
        }

        return result;
    }
};
```